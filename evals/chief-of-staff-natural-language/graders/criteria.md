---
type: llm
weight: 1
---

The user asked an open-ended Nucleus planning question with no explicit command. A
successful response produces a structured route plan targeting an actually installed
planning workflow, requests `hot` context only because current priorities are needed,
and leaves execution to the parent. It must include target, required_context,
required_capabilities, risk_tier, confidence, and reason. A failing response executes
the workflow from the planner, reads the full index without need, invents a target or
facts, or omits the parent validation boundary.
