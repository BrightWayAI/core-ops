---
description: Talk to your chief of staff. Natural-language front door for Nucleus — plans a route for Nucleus-domain or role-addressed work, then lets the parent validate and execute the selected installed workflow under the shared autonomy policy. Replaces nucleus-router.
---

# /cos

Resolve the installed plugin manifest set, then invoke the read-only
`chief-of-staff` agent (`agents/chief-of-staff.md`) with:

- the user's request verbatim;
- the installed workflow/role catalog;
- no memory contents unless entity or priority context is actually needed to
  disambiguate the route.

The agent returns a structured `route_plan`; it does not invoke the target. The
parent validates the plan against the installed catalog and executes it.

```yaml
route_plan:
  status: ready | ambiguous | unsupported
  intent: <short normalized intent>
  target:
    plugin: <installed plugin or null>
    workflow: <skill/command or null>
    role: <optional read-only role or null>
  required_context: [hot, entity-index, autonomy-policy, plugin-config]
  required_capabilities: [<logical capability names>]
  risk_tier: always | ask_first | never
  confidence: high | medium | low
  reason: <one sentence grounded in the catalog>
  question: <one disambiguating question only when status=ambiguous>
```

Execution rules:

1. `ambiguous` — the parent asks `question`; do not execute either candidate.
2. `unsupported` — name the missing plugin/capability and stop.
3. `ready` — verify the plugin and workflow are installed. Reject stale or
   invented targets.
4. Load only `required_context`: `hot` for current priorities, `entity-index`
   when a named person/client/project needs resolution, and the autonomy policy
   before any action. Do not load all of `hot.md` and `index.md` by default.
5. For `always`, narrate and execute. For `ask_first`, preview the exact mutation
   and confirm immediately before it. Never execute a `never` action.
6. Report the workflow run, skipped capabilities, evidence, and output location.

Plain natural language in a Nucleus session may reach this workflow without
`/cos`, but only when it contains a Nucleus domain, installed role, or known
workflow signal. Generic conversation and generic "help" remain outside its
trigger boundary.

If no plugin covers the request, the agent says so and names the missing capability rather than improvising a workaround.
