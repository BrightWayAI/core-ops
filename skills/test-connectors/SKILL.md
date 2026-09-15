---
disable-model-invocation: true
name: test-connectors
description: Run real, bounded, read-only connector integration probes and produce a privacy-safe Nucleus release report. Use when the user asks to test connectors, verify integrations, certify a release, run live connector smoke tests, or prove calendar, mail, CRM, Slack, Drive, enrichment, or transcript access works.
---

<!-- OPENAI-ADAPTER:START -->
## OpenAI host binding

Before acting, read `../../references/openai-portability.md`. That file translates
host-specific tools, agents, artifacts, scheduling, connectors, and config-root
access for ChatGPT and Codex. It overrides concrete Claude/Cowork tool names only;
the workflow, safety gates, and output contract in this skill remain canonical.
<!-- OPENAI-ADAPTER:END -->

Read `../../commands/test-connectors.md` completely and follow it as the canonical
workflow. A passing result requires an actual connector call. Preserve the read-only,
sandbox-target, bounded-query, and metadata-only evidence rules on every host.
