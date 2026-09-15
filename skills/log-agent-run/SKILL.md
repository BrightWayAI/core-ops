---
name: log-agent-run
description: Append a meta-record about a notable subagent run to ~/.brightway-state/agent-log.jsonl. Auto-fires on "/log-agent-run" or when a parent skill explicitly invokes this skill after an agent call. Captures agent / parent skill / confidence / user action — never message content or sensitive data.
---

<!-- OPENAI-ADAPTER:START -->
## OpenAI host binding

Before acting, read `../../references/openai-portability.md`. That file translates
host-specific tools, agents, artifacts, scheduling, connectors, and config-root
access for ChatGPT and Codex. It overrides concrete Claude/Cowork tool names only;
the workflow, safety gates, and output contract in this skill remain canonical.
<!-- OPENAI-ADAPTER:END -->


See `commands/log-agent-run.md` for full schema and behavior.
