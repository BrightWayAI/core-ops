---
type: llm
weight: 1
---

The user directly addressed a role ("Client Success") by its AI-staff display name
rather than typing a slash command or a plugin ID — this exercises the role-addressable
fallback. A successful response resolves "Client Success" to the `clients` plugin (which
owns the account after signature) and produces a structured route plan (target,
required_context, required_capabilities, risk_tier, confidence, reason) targeting its
weekly client-status surface for the Acme account, requesting only the Acme node/status
context it needs, and leaving execution to the parent. A failing response fails to
resolve the role name to the correct plugin, executes the status draft itself, or
fabricates Acme status details not present in memory.
