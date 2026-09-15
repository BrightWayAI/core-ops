---
disable-model-invocation: true
name: nucleus-dashboard
description: Deprecated alias for `/dashboard`, which was in turn moved to the `briefing` plugin as `briefing:dashboard` (2026-09-15). Only fires on explicit `/nucleus-dashboard` invocation, not on natural-language matching — use `briefing`'s `dashboard` skill for that.
---

<!-- OPENAI-ADAPTER:START -->
## OpenAI host binding

Before acting, read `../../references/openai-portability.md`. That file translates
host-specific tools, agents, artifacts, scheduling, connectors, and config-root
access for ChatGPT and Codex. It overrides concrete Claude/Cowork tool names only;
the workflow, safety gates, and output contract in this skill remain canonical.
<!-- OPENAI-ADAPTER:END -->


This skill's workflow moved to the `briefing` plugin's `dashboard` skill. See `briefing`'s
`skills/dashboard/SKILL.md` and `commands/dashboard.md` for the current workflow. `ops` retains
`/status` and `/diagnose`.

**If `briefing` is not installed:** skip this step. Note in the final output that the visual
dashboard isn't available, and suggest `/status` (terse text, still in `ops`) instead.

On explicit `/nucleus-dashboard` invocation, when `briefing` is installed: say once
"`/nucleus-dashboard` has moved to `briefing:dashboard` — routing you there now," then run the
`briefing:dashboard` skill's workflow unchanged.
