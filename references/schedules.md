# Schedule Library

_Edit this file to match your firm's standing schedules. `/register-schedules` reads it and registers each entry with Cowork's scheduled-tasks system._

_Instantiated from `schedules.template.md` on 2026-09-15 (Nucleus Operating Model Refactor Phase 1, step 1.4). Only `nightly-listen` is populated for now — add the rest as you adopt more of the marketplace._

## Schedule Table

| Name           | Cron / Schedule    | Action    | Plugin Required       | Owner | Notes | last_registered_id |
|----------------|---------------------|-----------|------------------------|-------|-------|---------------------|
| nightly-listen | daily 23:00 (11pm) | `/listen` | claude-cortex (v4.7+)  | self  | Nightly autonomous ingest, bound to `~/Documents/ClaudeCortex`. See "Registration prompt" below for the exact scheduled prompt and run-window guard. Output: `<config-root>/memory/staged/commit-drafts/YYYY-MM-DD.md` + refreshed `<config-root>/memory/hot.md`. Pair with `/morning` the next day. | _(pending — register in a Cowork session)_ |

## Registration prompt for `nightly-listen`

This is the prompt `/register-schedules` should hand to Cowork's scheduled-tasks tool for the `nightly-listen` row (per Nucleus Operating Model Refactor Phase 1 step 1.4). It describes only what the scheduled run should do — permission handling is a one-time setup step done by the human before registering (see First-run note below), not something the scheduled prompt itself should request.

```
Run window guard: if the current local time is after 06:00, exit immediately without
writing anything (stale trigger — e.g. the machine was asleep at the scheduled time
and Cowork is catching up late).

Otherwise, run the claude-cortex `/listen` command with no arguments (defaults to
"yesterday" in the identity time zone). Config root: ~/Documents/ClaudeCortex.
Read enabled sources from <config-root>/plugins/cortex.note-sources.md. On success,
this writes <config-root>/memory/staged/commit-drafts/YYYY-MM-DD.md and refreshes
<config-root>/memory/hot.md. This is an unattended run — do not prompt the user for
input; if a required tool has no prior authorization, log the gap and exit gracefully
rather than blocking.
```

**First-run note:** before relying on the schedule, run `/listen` once manually inside Cowork so each connector (calendar, Gmail, Slack, Drive, transcript sources per `cortex.note-sources.md`) gets its permission grant while a human is present to approve it. Cowork remembers per-tool grants across sessions, so the scheduled run afterward won't hit a blocking prompt. Do this deliberately, tool by tool — don't blanket-approve.

## How to read this file

- **Name** — kebab-case identifier. Used by `/register-schedules` to track which schedules have been registered (avoids duplicates).
- **Cron / Schedule** — cron expression OR human-friendly format. The register-schedules command translates either to Cowork's format.
- **Action** — the slash command or workflow to invoke.
- **Plugin Required** — must be installed for the schedule to work. Surface as a dependency check in `/register-schedules` and `/diagnose`.
- **Owner** — usually "self." Reserved for shared schedules (team scenarios) in future versions.
- **Notes** — purpose, edge cases, dependencies.
- **last_registered_id** — filled in by `/register-schedules` after a successful registration; prevents duplicate registration on re-run.

## After registration

After `/register-schedules` runs, the row's `last_registered_id` column gets filled in with Cowork's returned schedule ID, e.g. `cw-sched-abc123`.

## Adding more schedules later

See `schedules.template.md` for the full library of optional standing schedules (weekly-end-week, weekly-referrals, monthly-invoices, etc.) — copy rows in here as you adopt them. Per the Nucleus Operating Model Refactor, `daily-end-day` is intentionally NOT added here: `/end-day` is being demoted to optional (step 1.5), so it isn't a standing schedule by default.

## What this is NOT for

- One-off scheduled events. Those go directly into Cowork's scheduled-tasks panel.
- Calendar events for meetings. Use your calendar.
- External cron jobs (server-side). This file only governs Cowork-side scheduled tasks.
