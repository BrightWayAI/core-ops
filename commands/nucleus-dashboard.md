---
description: Deprecated — renamed to `/dashboard`, then moved to the `briefing` plugin as `briefing:dashboard`. Kept as a thin alias so existing invocations and schedules keep working.
---

# /nucleus-dashboard (deprecated)

This command was renamed to `/dashboard` as part of the 2026-09-15 `core-ops` → `ops` plugin rename, then moved out of `ops` entirely into the `briefing` plugin (Today's Brief) the same day, as part of consolidating every "what's going on" surface into `briefing`. `ops` still owns `/status` (terse text) and `/diagnose` (troubleshooting) — those did not move.

**If `briefing` is not installed:** skip this step. Note in the final output that the visual dashboard isn't available, and suggest `/status` (terse text, ops) as the closest available substitute.

Otherwise, say once: "`/nucleus-dashboard` has moved to `briefing:dashboard` — routing you there now."

Then run `briefing:dashboard` exactly as if the user had typed `/dashboard`, with no other behavior changes.
