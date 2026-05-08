---
description: Read the schedule library at references/schedules.md and register each schedule with Cowork's scheduled-tasks tool. Useful when setting up a new machine, recovering after a Cowork reinstall, or onboarding to the marketplace's standing schedules. Don't run this casually — it creates real scheduled tasks.
---

# /register-schedules

Bulk-register the standing schedules documented in `references/schedules.md` with Cowork's scheduled-tasks system. Replaces the manual "rerun every set-up step on the new machine" routine.

---

## Step 0 — Preflight

Verify `references/schedules.md` exists in this plugin (or wherever the user-context points). If missing, create from `references/schedules.template.md` and stop — tell user to populate first.

Verify the user has the scheduled-tasks tool available in Cowork. If not, the registration will fail; surface the dependency and stop.

---

## Step 1 — Read the schedule library

Parse `references/schedules.md`. Each entry is a markdown table row or section with these fields:

- **Name** — short identifier (kebab-case)
- **Cron** — cron expression (or "every N min" / "daily 9am" / similar)
- **Action** — the slash command or skill to invoke (e.g., `/end-day`, `/track-time`, `/referrals`)
- **Plugin dependency** — which plugin owns the action (so the user knows what must be installed)
- **Owner** — usually "self" but documented for clarity
- **Notes** — why this schedule exists, edge cases

Surface what's about to be registered:

```
## Schedules to register

| Name              | Cron            | Action               | Plugin       |
|-------------------|-----------------|----------------------|--------------|
| daily-end-day     | 0 17 * * 1-5    | /end-day             | cortex       |
| daily-track-time  | 0 18 * * 1-5    | /track-time          | time-tracking|
| weekly-end-week   | 0 16 * * 5      | /end-week            | cortex       |
| ...               | ...             | ...                  | ...          |

[N] schedules. Proceed? (y/n)
```

---

## Step 2 — Confirm

Wait for explicit user approval. Don't auto-register. Some schedules might have changed since the last machine setup; user should confirm the list reflects current intent.

If user says "modify," let them edit `references/schedules.md` and re-run.

---

## Step 3 — Register each

For each row in the confirmed list:
- Use the scheduled-tasks tool (e.g., `mcp__scheduled-tasks__create_scheduled_task`) to register the schedule
- Capture the returned schedule ID
- Append it to `references/schedules.md` as a `last_registered_id` field for that row (so future runs can see what's already registered and not duplicate)

If any registration fails, log the failure and continue with the rest. Surface failures at the end.

---

## Step 4 — Output

```
## Registration complete

✓ Successfully registered: [N]
- daily-end-day (id: ...)
- daily-track-time (id: ...)
- ...

✗ Failed: [N]
- [name] — reason: [error]

To verify: list schedules in Cowork's scheduled-tasks panel or run /list-schedules.
To unregister: edit references/schedules.md to remove rows, then run a future /unregister-schedules command (not yet implemented — manually unregister via Cowork for now).
```

---

## Behavior rules

- **Don't duplicate.** If a schedule with the same name was already registered (per `last_registered_id` annotation in schedules.md), skip it and note "already registered."
- **Confirm before writing.** Always show the list and wait for "y" before any registration calls.
- **Honor the user's edits.** If `references/schedules.md` was edited recently, treat the file as canonical — don't second-guess.
- **Surface failures cleanly.** Don't fail silently if one row errors. Continue with remaining rows.

---

## Common schedules

The plugin ships with a starter schedule library at `references/schedules.template.md`. Typical entries:

| Schedule | Why |
|---|---|
| Weekday 5pm `/end-day` | Daily reflection ritual |
| Friday 4pm `/end-week` | Weekly wrap-up |
| Friday 4pm `/track-time` (or weekday 6pm) | Daily/weekly time logging |
| Monday 6am `pipeline-analyst` (via wrapper skill) | Weekly pipeline ranking before the workday |
| Friday 7am `news-curator` (via wrapper skill) | Saturday-morning roundup prep |
| Friday 4pm `transcript-reviewer` (via wrapper skill) | Weekly commitment audit |
| Monthly 1st `/generate-invoices` | Monthly billing |
| Weekly Friday `/referrals` | Weekly referral digest |
| Weekly Friday `/client-status` | Weekly client status drafts |

Edit the schedule library to match your firm's rhythm.

---

## What this is NOT for

- One-off scheduled tasks. Use Cowork's scheduled-tasks panel directly.
- Scheduling things outside the marketplace (e.g., your morning standup). Use Cowork directly.
- Cross-machine sync. The schedule library lives in this plugin's repo, so it's already version-controlled. To sync schedules across machines: install the plugin → run `/register-schedules` → done.
