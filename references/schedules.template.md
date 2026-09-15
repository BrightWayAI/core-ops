# Schedule Library — Starter

_Copy this immutable starter to `<config-root>/plugins/ops/schedules.md`, then
customize the user-owned copy. The installed plugin directory is read-only at runtime._

## Schedule Table

| Name | Cron / Schedule | Action | Plugin Required | Owner | Required/Optional | Notes |
|---|---|---|---|---|---|---|
| nightly-listen | daily 23:00 (11pm) | `/listen` | cortex | self | Required | Unattended ingest with a post-06:00 stale-trigger guard. Stages proposals and refreshes `hot.md`; never writes durable nodes directly. |
| weekly-end-week | Friday 16:00 (4pm) | `/end-week` | cortex | self | Optional | Optional weekly review, cleanup, and reflection. |
| weekly-roundup | Friday 07:00 | `/roundup` | research | self | Optional — only relevant if research is installed | Stages cited roundup candidates; run comms's `/post` to draft. |

## Portability rules

- Keep definitions here free of scheduler IDs and machine names.
- `/register-schedules` stores per-host IDs under
  `<config-root>/plugins/ops/schedule-registrations/`.
- Every unattended prompt includes its run-window guard, permission behavior,
  expected output, and metadata-only receipt path.
- Remove or comment out schedules you do not use. Removing a row does not silently
  delete an already-registered host task.
