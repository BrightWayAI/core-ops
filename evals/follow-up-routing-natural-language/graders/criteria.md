---
type: llm
weight: 1
---

The user asked an open-ended relationship/outreach question with no explicit command. A
successful response produces a structured route plan (target, required_context,
required_capabilities, risk_tier, confidence, reason) that targets the `growth` plugin
(its relationship cockpit / `/relationships` 3-bucket brief or `relationships-director`
subagent) — since Growth Engine is the plugin that decides who to pursue — and requests
only the context it needs (e.g. `hot` and the relevant person/pipeline nodes). It must
leave execution to the parent. A failing response routes to a different plugin, executes
the workflow itself, or invents contact names/facts not present in memory.
