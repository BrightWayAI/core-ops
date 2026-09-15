# ops

A generic business-ops toolkit for Claude (Cowork + Claude Code).

The shared-utility plugin that other plugins in the [BrightWayAI marketplace](https://github.com/BrightWayAI/nucleus) lean on. Hosts the `chief-of-staff` natural-language front door, two CRM-intelligence subagents, and the cross-cutting infrastructure commands (`/diagnose`, telemetry, schedule library). Configurable per user via `/setup-core` — works with any CRM and any brand once configured.

## What's inside

### Agents (3)

- **`chief-of-staff`** — natural-language front door for the whole Nucleus stack. Loads `hot.md` + `index.md`, routes any request or role-addressed ask to the right installed command, narrates read-only work, and confirms before anything that writes, sends, or spends. Replaces the retired `nucleus-router` plugin.
- **`pipeline-analyst`** — point-in-time CRM ranking. Scores and ranks pipeline by recency × signal strength × stage, returning a prioritized weekly action list.
- **`pipeline-forecast`** — forward-looking revenue projection. Uses stage probabilities × deal values × historical close rates to forecast a window's revenue with sensitivity analysis (best / expected / worst / stretch), needle-mover deals, gap-to-target. Used monthly or pre-board-meeting.

### Slash commands (7)

- **`/cos`** — talk to your chief of staff. Natural-language front door; also fires implicitly on plain conversation via `skills/cos/SKILL.md` wherever the host supports always-loaded skill routing.
- **`/setup-core`** — interview that captures CRM context, brand context, and owner info. Writes to `<config-root>/plugins/ops.user-context.md`.
- **`/diagnose`** — ecosystem health check. Audits shared config files (`identity.md`, `voice.md`, cortex memory), plugin setup state, subagent availability, connector wiring. Produces a green/red checklist with specific fix instructions.
- **`/test-connectors`** — performs bounded, read-only calls against real authorized connectors and returns a sanitized release-certification report. User assertions and mocked payloads cannot pass.
- **`/log-agent-run`** — append a meta-record about a notable subagent invocation to `~/.brightway-state/agent-log.jsonl`. Captures agent / parent skill / confidence / user action — never message content.
- **`/agent-metrics`** — read-only digest of the agent log. Surfaces top performers, slipping agents, high-abandonment paths, confidence trends.
- **`/register-schedules`** — reconcile user-owned definitions from `<config-root>/plugins/ops/schedules.md` with the active host scheduler. Useful for new-machine setup or after scheduler reinstall.

## Install

Recommended: install via the [BrightWayAI marketplace](https://github.com/BrightWayAI/nucleus).

```
/plugin marketplace add BrightWayAI/nucleus
/plugin install ops@nucleus
```

## First-time setup

The plugin reads two shared user-level config files (created by cortex's `/setup-identity` and `/setup-voice`) before its own setup, so identity and voice questions don't get re-asked here.

Then run `/setup-core`. The interview captures:

- **CRM context** — which CRM you use, which pipeline stages matter, what "good" looks like for prioritization, optional historical close rates for forecasting.
- **Brand context** — brand colors, typography, tone-of-voice rules, optional path to your brand guide.
- **Owner info** — anything not already in shared identity.

Saved to `<config-root>/plugins/ops.user-context.md`.

You can re-run `/setup-core` anytime to update.

## Companion plugins

This plugin is a hub — many other plugins delegate to its subagents:

- **growth + briefing** call `pipeline-analyst` for weekly/daily prioritization.
- **Admin + Client Success** integrate with `pipeline-analyst` to surface revenue-vs-time and engagement-vs-pipeline views.
- All plugins benefit from `/diagnose` to verify setup state.
- `/register-schedules` orchestrates standing schedules across the whole marketplace.

## What's inside (file tree)

```
.claude-plugin/plugin.json     Plugin manifest
agents/
  chief-of-staff.md            Agent: natural-language front door (replaces nucleus-router)
  pipeline-analyst.md          Subagent: weekly CRM pipeline ranking
  pipeline-forecast.md         Subagent: forward-looking revenue projection
commands/
  cos.md                       Invoke the chief-of-staff agent
  setup-core.md                Interview and config writer
  diagnose.md                  Ecosystem health check
  test-connectors.md           Live, read-only connector certification
  log-agent-run.md             Telemetry: append to agent-log.jsonl
  agent-metrics.md             Telemetry: read-only log digest
  register-schedules.md        Bulk-register schedules from library
skills/
  cos/SKILL.md                 Always-loaded: routes free-form requests to the agent
  setup/SKILL.md               Auto-fires on setup phrases
  diagnose/SKILL.md            Auto-fires on diagnose phrases
  test-connectors/SKILL.md     Auto-fires on integration-test and release-certification phrases
  log-agent-run/SKILL.md       Auto-fires on log phrases
  agent-metrics/SKILL.md       Auto-fires on metrics phrases
  register-schedules/SKILL.md  Auto-fires on schedule phrases
references/
  user-context.template.md     Structure (committed)
  user-context.md              Your config (gitignored, created by setup)
  agent-log-schema.md          JSONL schema for the agent-run log
  schedules.template.md        Immutable starter schedule library
  schedules.md                 Immutable schema and workflow reference
```

## Dependencies

- **CRM connector** (HubSpot / Salesforce / Pipedrive / Attio / etc.) — Pipeline Analyst and Pipeline Forecast infer structure from properties.
- **File access** for Deliverable Reviewer — Drive connector or local path.
- **Cowork's scheduled-tasks tool** — required for `/register-schedules` to register schedules.

<!-- OPENAI-SUPPORT:START -->
## ChatGPT and Codex

Ops (Chief of Staff) ships as a native OpenAI plugin as well as a Claude plugin. In
ChatGPT desktop Local Work, enable **Chief of Staff** and ask naturally or mention
`@Chief of Staff`. In Codex, use natural language or the namespaced skills exposed
by the plugin. Claude slash-command names in this README remain workflow aliases.

All hosts resolve the same `<config-root>` used by Cortex, so Claude, ChatGPT desktop,
and Codex can share identity, voice, memory, and per-plugin settings without copying
them. The installed plugin directory is read-only at runtime. See
[`references/openai-portability.md`](references/openai-portability.md) for capability
mapping, connector checks, permissions, and honest degraded behavior.

Import the full catalog from
[`BrightWayAI/nucleus`](https://github.com/BrightWayAI/nucleus); Nucleus is the master
marketplace, while each plugin remains independently installable.
<!-- OPENAI-SUPPORT:END -->


## License

MIT. See `LICENSE`.
