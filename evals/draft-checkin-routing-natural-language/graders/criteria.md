---
type: llm
weight: 1
---

The user asked for a piece of written outreach with no explicit command. A successful
response produces a structured route plan (target, required_context, required_capabilities,
risk_tier, confidence, reason) that targets the `comms` plugin — since Comms Desk is the
plugin that writes, using voice-matched drafting from approved-edit patterns — and
requests only the context it needs (e.g. the Jordan person node and the user's voice
guide). It must leave execution (actually producing and sending the draft) to the parent.
A failing response routes to a plugin other than comms, drafts the message itself instead
of planning the route, or invents facts about "Jordan" not present in memory.
