---
disable-model-invocation: true
name: register-schedules
description: "Register user-owned standing schedules from <config-root>/plugins/core-ops/schedules.md with the active host scheduler, keeping per-host IDs and run receipts outside the installed plugin. Auto-fires on /register-schedules, register all my schedules, set up my standing schedules, rebuild my scheduled tasks, or similar provisioning requests. Common usage: first setup, a new machine, or host reinstall."
---

<!-- OPENAI-ADAPTER:START -->
## OpenAI host binding

Before acting, read `../../references/openai-portability.md`. That file translates
host-specific tools, agents, artifacts, scheduling, connectors, and config-root
access for ChatGPT and Codex. It overrides concrete Claude/Cowork tool names only;
the workflow, safety gates, and output contract in this skill remain canonical.
<!-- OPENAI-ADAPTER:END -->


See `commands/register-schedules.md` for the full workflow.
