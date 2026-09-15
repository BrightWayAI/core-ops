---
disable-model-invocation: true
name: register-schedules
description: "Bulk-register the standing schedules documented in references/schedules.md with the host scheduler. Auto-fires on /register-schedules, register all my schedules, set up my standing schedules, register the schedule library, rebuild my scheduled tasks, or any phrase about provisioning the marketplace's standard scheduled tasks. Common usage: new machine setup or host reinstall."
---

<!-- OPENAI-ADAPTER:START -->
## OpenAI host binding

Before acting, read `../../references/openai-portability.md`. That file translates
host-specific tools, agents, artifacts, scheduling, connectors, and config-root
access for ChatGPT and Codex. It overrides concrete Claude/Cowork tool names only;
the workflow, safety gates, and output contract in this skill remain canonical.
<!-- OPENAI-ADAPTER:END -->


See `commands/register-schedules.md` for the full workflow.
