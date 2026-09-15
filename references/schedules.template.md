# Schedule Library — Starter

_Copy this immutable starter to `<config-root>/plugins/ops/schedules.md`, then
customize the user-owned copy. The installed plugin directory is read-only at runtime._

## Schedule Table

| Name | Cron / Schedule | Action | Plugin Required | Owner | Notes |
|---|---|---|---|---|---|
| nightly-listen | daily 23:00 (11pm) | `/listen` | cortex | self | Unattended ingest with a post-06:00 stale-trigger guard. Stages proposals and refreshes `hot.md`; never writes durable nodes directly. |
| weekly-end-week | Friday 16:00 (4pm) | `/end-week` | cortex | self | Optional weekly review, cleanup, and reflection. |
| weekly-relationships | Monday 06:00 | `/relationships` | growth | self | Prepares the week's prioritized relationship actions; never sends. |
| weekly-news-roundup | Friday 07:00 | `/roundup` | research | self | Stages cited roundup candidates; run comms's `/post` (or `/roundup --draft` if comms is installed) to draft. |
| weekly-client-status | Friday 14:00 | `/client-status` | clients | self | Produces reviewable client-status drafts; never sends. |
| weekly-research-gaps | Saturday 09:00 | `/research-gaps` | cortex | self | Stages cited memory-gap proposals for review. |
| monthly-invoices | 1st of month 09:00 | `/generate-invoices` | admin | self | Generates invoice drafts for the prior month; never sends. |
| monthly-pipeline-forecast | 1st of month 10:00 | `growth:pipeline-forecast` | growth | self | Produces a read-only forecast and gap-to-target analysis. |
| monthly-cleanup | 1st of month 17:00 | `/cleanup` | cortex | self | Audits memory health; destructive actions remain confirmation-gated. |

`/end-day` is intentionally omitted from the default standing schedules. It remains
an optional reflection ritual; nightly `/listen` plus `/morning` is the primary loop.

## Portability rules

- Keep definitions here free of scheduler IDs and machine names.
- `/register-schedules` stores per-host IDs under
  `<config-root>/plugins/ops/schedule-registrations/`.
- Every unattended prompt includes its run-window guard, permission behavior,
  expected output, and metadata-only receipt path.
- Remove or comment out schedules you do not use. Removing a row does not silently
  delete an already-registered host task.
