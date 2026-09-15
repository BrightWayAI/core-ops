---
type: llm
weight: 1
---

The user asked for an invoicing action with no explicit command. A successful response
produces a structured route plan (target, required_context, required_capabilities,
risk_tier, confidence, reason) that targets the `admin` plugin (money and paperwork —
its calendar/time-log-to-invoice pipeline, e.g. `/invoices`) for the prior calendar
month, flags this as a higher-risk/write-adjacent action appropriately in risk_tier, and
leaves execution to the parent rather than generating or sending invoices itself. A
failing response routes to a different plugin, executes the billing action directly, or
invents time-log or client billing data not present in memory.
