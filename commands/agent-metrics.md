---
description: Summarize the agent-run log to surface patterns over time — which agents are pulling weight, which paths bleed into manual takeover, where confidence routinely lands Low. Read-only over the log; never modifies it. Run monthly or whenever you suspect an agent is slipping.
---

# /agent-metrics

Read the agent-run log at `~/.brightway-state/agent-log.jsonl` and produce a digest of patterns. Helps you decide which agents to refine, which to scrap, which are quietly compounding.

---

## Step 0 — Preflight

Read `~/.brightway-state/agent-log.jsonl`. If missing or empty, say:
> "No agent-run log yet. The log gets populated when parent skills invoke `/log-agent-run` after notable agent runs. Start logging by adding the call to skills you care about."

If present, parse all JSON Lines records.

---

## Step 1 — Determine the window

Default: last 90 days. User can override (`/agent-metrics --since 2026-01-01`).

Filter records to the window.

---

## Step 2 — Compute per-agent stats

For each agent that appeared in the window:

| Metric | Definition |
|---|---|
| `invocations` | Count of records |
| `confidence_distribution` | % High / Medium / Low |
| `user_action_distribution` | % accepted / iterated / overrode / abandoned |
| `acceptance_rate` | % records where user_action == accepted |
| `abandonment_rate` | % where user_action == abandoned |
| `avg_latency_seconds` | If latency is captured in records |
| `top_parent_skills` | Top 3 parent skills that invoked this agent |

---

## Step 3 — Compute per-skill stats

For each parent_skill that invoked any agent:

| Metric | Definition |
|---|---|
| `agent_calls` | Count of agent invocations from this skill |
| `agents_used` | Distinct agents called by this skill |
| `iteration_rate` | % where user_action == iterated (suggests skill-output is close-but-not-quite) |
| `abandonment_rate` | % where user_action == abandoned |

---

## Step 4 — Identify patterns

Surface notable findings:

- **Top performer** — agent with highest acceptance rate AND >5 invocations. "[agent] is your strongest — keep relying on it."
- **Slipping agent** — agent whose acceptance rate dropped >20% in the last 30 days vs. the prior 60. "[agent] used to land 80% of the time; last 30 days it's 55%. Worth a look."
- **High-abandonment paths** — parent_skill with abandonment_rate > 30%. "[skill] sees frequent manual takeovers — the agent it relies on may not be matching the workflow."
- **Confidence drift** — agent whose Low-confidence rate is climbing. "[agent] is increasingly returning Low confidence — input quality may be degrading (stale CRM data, broken connector, etc.)."
- **Underused agent** — agent with <3 invocations in window. "[agent] is barely used — either you don't need it or it's not being routed to."

---

## Step 5 — Output the digest

```
## Agent Metrics — [window]

### Top performers (high acceptance, high volume)
- **[agent]** — [N] invocations, [X]% accepted. Used most by: [skill1], [skill2].

### Slipping agents (acceptance dropped recently)
- **[agent]** — [old_rate]% → [new_rate]%. Likely cause: [hypothesis if signals point one way].

### High-abandonment paths (workflow may be off)
- **[skill]** — [N] agent calls, [X]% abandoned. Worth investigating: is the agent being asked to do something it can't, or is the user-context stale?

### Confidence trends
- **[agent]** — Low confidence is up [X]% over last 30d. Possible: [hypothesis].

### Underused (consider retiring or routing more)
- **[agent]** — only [N] invocations in [window].

### Volume summary
| Agent | Invocations | Acceptance | Avg Confidence |
|-------|-------------|------------|----------------|
| ...   | ...         | ...        | ...            |

### Recommendations
1. [Specific action — e.g., "Investigate [agent]'s Low-confidence rate; rerun /diagnose to check CRM connection"]
2. [Specific action]
3. [Specific action]
```

---

## Behavior rules

- **Don't speculate beyond the data.** If acceptance rate is dropping, say so. Don't invent reasons unless the records support a clear pattern.
- **Be honest about sample size.** With <10 invocations, any % is noise. Flag low-volume agents with a "small sample" warning rather than computing precise rates.
- **Surface the most actionable thing first.** Don't dump 20 metrics equally; rank by what the user might actually do something about.
- **Never modify the log.** Read-only. The log is append-only by design.

---

## When to run this

- **Monthly** — quick health check on the agent ecosystem
- **After a noticeable change** — you swapped an agent's spec, restarted Cowork, switched CRMs — see if metrics shifted
- **Before deprecating an agent** — confirm via data that an agent is or isn't pulling weight before you decide to remove it
- **When something feels off** — gut check before debugging

Don't run more than weekly — variance dominates short windows and the metrics become noise.

---

## Privacy

The log itself is meta-only (no contact names, no message content). The metrics digest produced is also meta-only. Safe to share with a coach, peer, or write up as part of a quarterly review of your stack.
