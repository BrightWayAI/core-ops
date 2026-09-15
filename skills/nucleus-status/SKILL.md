---
disable-model-invocation: true
name: nucleus-status
description: Deprecated alias for `/status` (renamed 2026-09-15). Only fires on explicit `/nucleus-status` invocation, not on natural-language matching — use `status` for that.
---

<!-- OPENAI-ADAPTER:START -->
## OpenAI host binding

Before acting, read `../../references/openai-portability.md`. That file translates
host-specific tools, agents, artifacts, scheduling, connectors, and config-root
access for ChatGPT and Codex. It overrides concrete Claude/Cowork tool names only;
the workflow, safety gates, and output contract in this skill remain canonical.
<!-- OPENAI-ADAPTER:END -->


This skill was renamed to `status`. See `skills/status/SKILL.md` and `commands/status.md`
for the current workflow.

On explicit `/nucleus-status` invocation: say once "`/nucleus-status` has been renamed to
`/status` — routing you there now," then run the `status` skill's workflow unchanged.
