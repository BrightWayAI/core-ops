---
name: agent-metrics
description: Summarize the agent-run log to surface patterns. Auto-fires on "/agent-metrics", "how are my agents doing", "agent performance", "which subagents are working", "audit agent quality", "agent log digest", or any phrase about subagent quality / acceptance / abandonment. Read-only over ~/.brightway-state/agent-log.jsonl. Run monthly.
---

<!-- OPENAI-ADAPTER:START -->
## OpenAI host binding

Before acting, read `../../references/openai-portability.md`. That file translates
host-specific tools, agents, artifacts, scheduling, connectors, and config-root
access for ChatGPT and Codex. It overrides concrete Claude/Cowork tool names only;
the workflow, safety gates, and output contract in this skill remain canonical.
<!-- OPENAI-ADAPTER:END -->


See `commands/agent-metrics.md` for the full digest workflow.
