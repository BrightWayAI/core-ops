# Schedule Library

_Edit this file to match your firm's standing schedules. `/register-schedules` reads it and registers each entry with Cowork's scheduled-tasks system._

The defaults below match the most common patterns for users running the full BrightWayAI marketplace stack. Customize as needed — comment out, delete, add custom rows.

## Schedule Table

| Name                  | Cron / Schedule         | Action                       | Plugin Required          | Owner | Notes                                                                |
|-----------------------|-------------------------|------------------------------|--------------------------|-------|----------------------------------------------------------------------|
| nightly-listen        | daily 23:00 (11pm)      | /listen                      | claude-cortex (v4.7+)    | self  | Nightly autonomous ingest. Archives yesterday's calendar/inbox/slack/transcripts. Stages /morning proposals. |
| daily-end-day         | weekday 17:00 (5pm)     | /end-day                     | claude-cortex            | self  | Daily reflection ritual. Captures wins/blocks/learnings to memory.   |
| daily-track-time      | weekday 18:00 (6pm)     | /track-time                  | time-tracking            | self  | Pull yesterday's calendar, classify billable time. ~5 min daily.     |
| weekly-end-week       | Friday 16:00 (4pm)      | /end-week                    | claude-cortex            | self  | Friday wrap-up: transcript-reviewer, cleanup, review, reflection.    |
| weekly-pipeline       | Monday 06:00            | /weekly-outreach pre-stage   | weekly-outreach          | self  | Monday morning: weekly outreach plan ready for review.               |
| weekly-news-roundup   | Friday 07:00            | /ai-roundup pre-stage        | news-curator             | self  | Friday: candidate list ready for afternoon drafting.                 |
| weekly-transcripts    | Friday 16:00            | (via /end-week)              | claude-cortex            | self  | Surfaces uncaptured commitments. Triggered inside /end-week.         |
| weekly-referrals      | Friday 14:00            | /referrals                   | referral-engine          | self  | Weekly referral digest — top 3 actions for the week.                 |
| weekly-client-status  | Friday 14:00            | /client-status               | client-status            | self  | Drafts weekly client updates per active engagement.                  |
| weekly-research-gaps  | Saturday 09:00          | /research-gaps               | claude-cortex (v4.5+)    | self  | Weekly memory-gap scan. Stages draft for review.                     |
| monthly-invoices      | 1st of month 09:00      | /generate-invoices           | time-tracking            | self  | Generate invoices for prior month. Review and send.                  |
| monthly-pipeline-fcst | 1st of month 10:00      | /forecast                    | core-ops                 | self  | Monthly pipeline forecast for runway / target tracking.              |
| monthly-cleanup       | 1st of month 17:00      | /cleanup                     | claude-cortex            | self  | Memory hygiene audit.                                                |

## How to read this file

- **Name** — kebab-case identifier. Used by `/register-schedules` to track which schedules have been registered (avoids duplicates).
- **Cron / Schedule** — cron expression OR human-friendly format. The register-schedules command translates either to Cowork's format.
- **Action** — the slash command or workflow to invoke. Some actions are "via /end-week" because they're orchestrated by a parent — the parent gets the cron, not the orchestrated thing.
- **Plugin Required** — must be installed for the schedule to work. Surface as a dependency check in `/register-schedules` and `/diagnose`.
- **Owner** — usually "self." Reserved for shared schedules (team scenarios) in future versions.
- **Notes** — purpose, edge cases, dependencies.

## After registration

After `/register-schedules` runs, each row gets an annotation:

```
| daily-end-day | weekday 17:00 | /end-day | claude-cortex | self | Daily ritual. last_registered_id: cw-sched-abc123 |
```

This prevents duplicate registration on re-run.

## Customizing

Common edits:
- **Change times to match your time zone and rhythm** — defaults are US-business-hours-friendly
- **Disable schedules you don't want** — comment out the row or delete it
- **Add custom workflows** — e.g., `Sunday 19:00` `/recall company-ops` for weekly company-state snapshot
- **Per-machine variation** — if you have a laptop and a desktop with different rhythms, fork this file per machine

## What this is NOT for

- One-off scheduled events. Those go directly into Cowork's scheduled-tasks panel.
- Calendar events for meetings. Use your calendar.
- External cron jobs (server-side). This file only governs Cowork-side scheduled tasks.

## Migrating between machines

To set up the same schedule library on a new machine:

1. Install core-ops via the marketplace
2. Confirm `references/schedules.md` matches what you want (it's version-controlled in the plugin repo, so it follows the install)
3. Run `/register-schedules` — Cowork prompts for confirmation, then registers each
4. Done — your new machine has the same standing schedules as the old one

This is the entire reason the schedule library exists: schedules used to live per-machine in Cowork's local state. Now they're a versioned plugin artifact.
