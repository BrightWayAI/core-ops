---
description: Register user-owned schedule definitions from the shared config root with an available host scheduler. Keeps immutable plugin defaults separate from per-host registration IDs and run receipts. Useful on first setup, a new machine, or after a scheduler reinstall. Creates real scheduled tasks and always requires confirmation.
---

# /register-schedules

Provision standing schedules from user-owned definitions. The installed plugin is
read-only at runtime: bundled references are defaults, never mutable state.

## State contract

- Definitions: `<config-root>/plugins/ops/schedules.md`
- Bundled starter: `references/schedules.template.md`
- Per-host registration state:
  `<config-root>/plugins/ops/schedule-registrations/<host-id>.json`
- Metadata-only run receipts:
  `<config-root>/plugins/ops/schedule-runs/<schedule>/<run-id>.json`

`<host-id>` is a stable, non-secret identifier such as
`cowork-macbook-pro` or `codex-desktop`. Never put host-specific IDs in the
definition file or installed plugin directory.

## Step 0 — Resolve and initialize

Before doing anything else, tell the user two things: (1) the computer must be awake
(not asleep) at the scheduled run time for an unattended task to fire, and (2) the
first unattended run will prompt for tool-use approvals — the user should approve
each with "always allow" so later runs don't stall waiting on a prompt nobody is
watching.

Resolve `<config-root>` using the shared precedence chain: explicit override,
`CORTEX_CONFIG_ROOT`, `~/.cortex/config-root` (primary), legacy pointer
`~/Documents/.claude-plugin-config-root` (fallback), then default.

If the definitions file is missing, preview the bundled starter and offer to copy it
to the definitions path. If the user declines, return the path and stop. Never edit
`references/schedules.template.md` or `references/schedules.md` at runtime.

By default, only register `nightly-listen` — it is the one required schedule.
`weekly-end-week` and `weekly-roundup` (the latter only if `research` is installed)
are available in the starter but not auto-registered; mention them and offer to
register either if the user wants them.

Detect `scheduler.register`. If unavailable, validate and return the definitions for
manual setup without claiming registration.

## Step 1 — Validate definitions and dependencies

Parse each enabled row:

- `name` — unique kebab-case identifier
- `schedule` — cron or host-readable schedule
- `action` — installed workflow to invoke
- `plugin` — owning plugin
- `owner` — normally `self`
- `notes` — guardrails and expected output

Reject duplicate names, missing fields, retired plugins/actions, or dependencies that
are not installed. A missing optional plugin skips that row; it does not invalidate
the rest of the library.

Compute a definition fingerprint from normalized `name + schedule + action + plugin`.
This lets registration state detect changed definitions without coupling state to
Markdown formatting.

## Step 2 — Reconcile with this host

Read this host's registration-state file if present. When the scheduler can list
existing tasks, reconcile against live state; live state wins over the cache.

Classify each definition:

- `new` — no matching live task or state record
- `current` — live task and fingerprint match
- `changed` — name matches but fingerprint differs
- `missing-live` — cached ID exists but scheduler no longer has it
- `unavailable` — dependency or scheduler capability missing

Never skip solely because a cached ID exists. Never register a duplicate solely
because a state file is absent.

## Step 3 — Preview and confirm

Show new, changed, skipped, and current schedules, including exact action and timing.
Ask once for explicit confirmation before any create/update call. Changed schedules
must state whether the host will update in place or replace the old task.

## Step 4 — Register and persist state

For each confirmed `new`, `changed`, or `missing-live` entry:

1. Register through the active host scheduler.
2. Capture returned ID and scheduler name.
3. Atomically update only this host's registration-state JSON.
4. Continue after individual failures and record a sanitized error code.

State schema:

```json
{
  "schema_version": "1.0.0",
  "host_id": "cowork-macbook-pro",
  "scheduler": "cowork",
  "updated_at": "2026-09-15T18:00:00-04:00",
  "registrations": {
    "nightly-listen": {
      "scheduler_id": "opaque-host-id",
      "definition_fingerprint": "sha256:...",
      "registered_at": "2026-09-15T18:00:00-04:00",
      "last_verified_at": "2026-09-15T18:00:00-04:00"
    }
  }
}
```

Write atomically. Schedule IDs are machine-specific operational metadata, not source
configuration and not portable between hosts.

## Step 5 — Scheduled-run receipts

Every registered prompt must request one metadata-only receipt per run. It contains:

```json
{
  "schema_version": "1.0.0",
  "run_id": "<host run id or UUID>",
  "schedule": "nightly-listen",
  "host_id": "cowork-macbook-pro",
  "started_at": "<ISO-8601>",
  "ended_at": "<ISO-8601>",
  "outcome": "success|partial|skipped|failed",
  "source_coverage": {"calendar": "read|skipped|failed"},
  "outputs": [{"path": "<config-root-relative path>", "sha256": "..."}],
  "error_codes": []
}
```

Receipts contain no connector payloads, messages, transcript text, credentials, or
memory content. If the target workflow cannot write a receipt on the active host, say
so during preview and rely on host scheduler history instead.

## Output

Report counts for current, registered, updated, skipped, and failed entries; list IDs
only for the current host; and state where registration state and run receipts live.
Do not claim success for rows that were merely returned as manual definitions.

## Behavior rules

- Installed plugin directories are immutable at runtime.
- Definitions are portable; registration state is per-host.
- Live scheduler state wins over cached IDs.
- Registration and changes always require confirmation.
- Failures are explicit and do not prevent independent rows from continuing.
- Removing a definition does not silently unregister its live task; surface it as an
  orphan and ask before deletion.

## What this is not for

- One-off reminders or meetings.
- External cron provisioning when the active host exposes no scheduler.
- Automatic unregistration or destructive scheduler cleanup.
