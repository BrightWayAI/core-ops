---
type: llm
weight: 1
---

The user asked an open-ended natural-language question with no explicit command and no named plugin/role. A successful response recognizes this as a chief-of-staff routing request: it loads or references available context (hot.md/index.md if cortex is present) and either routes to/narrates the matching installed workflow, or plainly names what capability is missing if nothing installed covers it — never inventing a workaround or a fabricated answer. A failing response gives a generic unhelpful reply, ignores the request, or fabricates specific facts (meetings, tasks, deals) it has no source for.
