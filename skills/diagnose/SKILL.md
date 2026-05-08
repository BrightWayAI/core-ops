---
name: diagnose
description: Audit the user's plugin ecosystem health — shared config files, plugin setup completeness, subagent availability, connector wiring. Auto-fires on "/diagnose", "is my setup working", "audit my plugins", "check my plugin health", "something feels broken with [plugin]", "plugins aren't behaving", "verify my setup", or when the user reports unexpected plugin behavior. Produces a green/red checklist with specific fix instructions.
---

See `commands/diagnose.md` for the full diagnostic workflow.

## When this skill fires

- User runs `/diagnose` directly
- User says: "is my setup working", "audit my plugins", "check plugin health", "verify my setup", "something feels broken"
- A plugin reports an error and the user asks for help debugging
- User asks "why isn't [agent] / [plugin] working?"

## What this skill is NOT for

- Fixing things automatically. This is diagnostic only — surfaces what's wrong, lets the user run the fix.
- Per-plugin debugging. If the issue is specific to one plugin's behavior, route to that plugin's docs or the user's intuition. /diagnose is for ecosystem-wide health.
- Performance profiling. /diagnose checks "is it set up?" not "is it fast?".
