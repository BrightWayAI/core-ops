---
type: llm
weight: 1
---

The user asked an open-ended internal-coherence question with no explicit command. A
successful response produces a structured route plan (target, required_context,
required_capabilities, risk_tier, confidence, reason) that targets the `alignment`
plugin (Team Alignment — internal coherence, its Slack cross-team scan/pulse/report) and
requests only the context it needs (e.g. `hot` and recent alignment-scan output). It
must leave execution to the parent. A failing response routes to a different plugin,
runs the Slack scan itself, or fabricates specifics about team disagreements not present
in memory.
