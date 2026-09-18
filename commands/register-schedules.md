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

Before doing anything else, tell the user three things: (1) the computer must be
awake (not asleep) at the scheduled run time for an unattended task to fire, (2) the
first unattended run will prompt for tool-use approvals — the user should approve
each with "always allow" so later runs don't stall waiting on a prompt nobody is
watching, and (3) **every schedule registered here is bound to this specific
computer** — the task fires from the Mac that this registering conversation is
linked to, and that Mac must be online with the Claude desktop app running at fire
time, or the run is silently skipped until the next scheduled window.

Resolve `<config-root>` using the shared precedence chain: explicit override,
`CORTEX_CONFIG_ROOT`, `~/.cortex/config-root` (primary), legacy pointer
`~/Documents/.claude-plugin-config-root` (fallback), then default.

Every schedule registered by this command reads or writes `<config-root>` — there is
no schedule in the library today that doesn't. Accordingly, every registration in
Step 4 below MUST request `requires_local_device=true` and
`folders=[<config-root absolute path>]` (the resolved absolute path, not a
placeholder). This is what makes "Require this computer" + the folder attachment
happen at creation time instead of leaving the task able to fire on a device that
can never reach `<config-root>`.

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

Compute a definition fingerprint from normalized
`name + schedule + action + plugin + requires_local_device + folders`. Including the
device-binding fields means an already-registered task that predates this
requirement (fingerprint computed without them) classifies as `changed` on the next
run and gets re-registered with proper folder binding automatically — this is the
self-healing path for existing installs, including the live `nightly-listen` task
registered before this contract existed.

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

1. Call `create_trigger` (or the host's equivalent) with `requires_local_device=true`
   and `folders=[<config-root>]` set explicitly — never omit these fields, even for
   `changed` entries that are only being re-registered to pick up the binding.
2. Capture returned ID and scheduler name.
3. **Verify the binding actually took.** Call `list_triggers`, find the just-created
   or just-updated task by its returned id, and require both:
   - `derived_state.folders_state != NONE`
   - the resolved `<config-root>` absolute path is present in `derived_state.folders`
   If either check fails, do **not** write a registration-state record for this
   entry. Report it as `failed: no-folder-binding` with this exact instruction:
   "open the task in the Claude desktop app on the linked Mac, turn on 'Require this
   computer', attach `<config-root>`, then re-run `/register-schedules --verify`."
   Treat this the same as any other per-row failure — continue reconciling the rest
   of the library.
4. On a successful verification, atomically update only this host's
   registration-state JSON, and stamp `last_verified_at` to now.
5. Continue after individual failures and record a sanitized error code.

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
      "requires_local_device": true,
      "folders": ["<config-root absolute path>"],
      "registered_at": "2026-09-15T18:00:00-04:00",
      "last_verified_at": "2026-09-15T18:00:00-04:00"
    }
  }
}
```

Write atomically. Schedule IDs are machine-specific operational metadata, not source
configuration and not portable between hosts.

## Step 4.5 — `--verify` mode

`/register-schedules --verify` re-runs only the Step 4 folder-binding checks against
this host's already-registered live tasks — it never creates, updates, or deletes a
task. For each registration in this host's state file:

1. `list_triggers`, find the task by its stored `scheduler_id`.
2. Not found live → classify `missing-live`; do not update `last_verified_at`; report
   it in the output so a plain `/register-schedules` run can re-create it.
3. Found, but `derived_state.folders_state == NONE` or `<config-root>` absent from
   `derived_state.folders` → leave the registration-state record as-is (don't erase
   history) but report `failed: no-folder-binding` with the same repair instruction
   as Step 4, and do NOT bump `last_verified_at`.
4. Found and bound correctly → update only `last_verified_at` for that entry.

This mode is read-only against the scheduler aside from the `list_triggers` call and
the local `last_verified_at` bump — no `create_trigger`/`update_trigger` call is ever
made in `--verify`. `ops`'s `/status` command calls this mode read-only (it does not
invoke registration) to render the SCHEDULED LOOP section.

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

Every registered prompt must keep the existing run-window guard (skip a catch-up run
outside its intended window rather than firing stale) and the receipt requirement
above, and MUST additionally include this instruction verbatim as part of what gets
registered: "If `<config-root>` is not readable, write nothing, emit a receipt with
`status=failed` and `error_code=config_root_unreachable`, and exit non-zero. Do not
report success." This is what turns an unreachable config root (e.g. a task that
fired on the wrong device, or a Mac where the folder was detached) into a loud,
visible failure instead of a scheduler-reported "success" that did nothing.

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
