---
name: setup
description: Configure core-ops for your CRM, brand, and company context. Auto-fires on "set up core-ops", "configure pipeline analysis", "set up deliverable review", "/setup-core", or any phrase about getting core-ops ready to use. Also fires when another core-ops skill or agent reports that user-context.md is missing or empty.
---

See `commands/setup-core.md` for the full interview workflow.

## When this skill fires automatically

- User runs `/setup-core` directly
- User says: "set up core-ops", "configure core-ops", "set up pipeline analyst", "set up deliverable review", "configure my CRM context"
- User installs the plugin for the first time and asks "how do I use this?"
- Another core-ops skill or agent reports that `references/user-context.md` is missing — auto-route here

## Quick path

If the user just wants to skip the interview and use defaults: write a minimal `references/user-context.md` with placeholders, note that `/pipeline-analyst` and `/review-deliverable` will work in degraded mode (default scoring weights, generic brand checks), and tell them to re-run `/setup-core` whenever they're ready.

Don't do this silently — only on explicit request. The interview is short and the agents are noticeably better with real context.
