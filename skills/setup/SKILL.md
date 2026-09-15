---
disable-model-invocation: true
name: setup
description: Configure ops for your CRM, brand, and company context. Auto-fires on "set up ops", "configure pipeline analysis", "set up deliverable review", "/setup-core", or any phrase about getting ops ready to use. Also fires when another ops skill or agent reports that user-context.md is missing or empty.
---

<!-- OPENAI-ADAPTER:START -->
## OpenAI host binding

Before acting, read `../../references/openai-portability.md`. That file translates
host-specific tools, agents, artifacts, scheduling, connectors, and config-root
access for ChatGPT and Codex. It overrides concrete Claude/Cowork tool names only;
the workflow, safety gates, and output contract in this skill remain canonical.
<!-- OPENAI-ADAPTER:END -->


See `commands/setup-core.md` for the full interview workflow.

## When this skill fires automatically

- User runs `/setup-core` directly
- User says: "set up ops", "configure ops", "set up pipeline analyst", "set up deliverable review", "configure my CRM context"
- User installs the plugin for the first time and asks "how do I use this?"
- Another ops skill or agent reports that `<config-root>/plugins/ops.user-context.md` is missing — auto-route here

## Quick path

If the user just wants to skip the interview and use defaults: write a minimal `<config-root>/plugins/ops.user-context.md` with placeholders, note that growth's `pipeline-analyst` and clients' `/review-deliverable` will work in degraded mode (default scoring weights, generic brand checks), and tell them to re-run `/setup-core` whenever they're ready.

Don't do this silently — only on explicit request. The interview is short and the agents are noticeably better with real context.
