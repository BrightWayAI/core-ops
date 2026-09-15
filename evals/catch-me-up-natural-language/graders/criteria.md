---
type: llm
weight: 1
---

The user asked an open-ended "what's the state of this account" question with no
explicit command and no @-mention of a specific plugin. A successful response produces
a structured route plan (target, required_context, required_capabilities, risk_tier,
confidence, reason) targeting an actually installed status/recall surface for the named
entity — this is a cross-plugin lookup, so the target may be cortex recall for the
`Acme` node, `clients`'s status surface, or `briefing`'s review/timeline depending on
what's installed and what the entity resolves to — and requests only the context needed
(e.g. the Acme node or its latest status). It must leave execution to the parent rather
than running the lookup itself. A failing response executes the workflow directly from
the planner, invents facts about "Acme" that aren't in memory, or fabricates a target
plugin that doesn't exist.
