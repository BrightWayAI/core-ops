# core-ops

A generic business-ops toolkit for Claude (Cowork + Claude Code).

The shared-utility plugin that other plugins in the [BrightWayAI marketplace](https://github.com/BrightWayAI/claude-plugins) lean on. Hosts two subagents (CRM intelligence) and the cross-cutting infrastructure commands (`/diagnose`, telemetry, schedule library). Configurable per user via `/setup-core` — works with any CRM and any brand once configured.

## What's inside

### Subagents (2)

- **`pipeline-analyst`** — point-in-time CRM ranking. Scores and ranks pipeline by recency × signal strength × stage, returning a prioritized weekly action list. Used by `weekly-outreach`, `plan-tomorrow`, any pipeline-review workflow.
- **`pipeline-forecast`** — forward-looking revenue projection. Uses stage probabilities × deal values × historical close rates to forecast a window's revenue with sensitivity analysis (best / expected / worst / stretch), needle-mover deals, gap-to-target. Used monthly or pre-board-meeting.

### Slash commands (5)

- **`/setup-core`** — interview that captures CRM context, brand context, owner info. Writes to `references/user-context.md`.
- **`/review-deliverable [path]`** — QA pass on a client deliverable (deck, doc, spreadsheet, one-pager) against your brand guide and the original brief. Returns location-tagged findings ranked by severity, plus a ship/no-ship verdict.
- **`/diagnose`** — ecosystem health check. Audits shared config files (`identity.md`, `voice.md`, cortex memory), plugin setup state, subagent availability, connector wiring. Produces a green/red checklist with specific fix instructions.
- **`/log-agent-run`** — append a meta-record about a notable subagent invocation to `~/.brightway-state/agent-log.jsonl`. Captures agent / parent skill / confidence / user action — never message content.
- **`/agent-metrics`** — read-only digest of the agent log. Surfaces top performers, slipping agents, high-abandonment paths, confidence trends.
- **`/register-schedules`** — bulk-register standing schedules from `references/schedules.md` with Cowork's scheduled-tasks system. Useful for new-machine setup or after Cowork reinstall.

## Install

Recommended: install via the [BrightWayAI marketplace](https://github.com/BrightWayAI/claude-plugins).

```
/plugin marketplace add BrightWayAI/claude-plugins
/plugin install core-ops@claude-plugins
```

## First-time setup

The plugin reads two shared user-level config files (created by cortex's `/setup-identity` and `/setup-voice`) before its own setup, so identity and voice questions don't get re-asked here.

Then run `/setup-core`. The interview captures:

- **CRM context** — which CRM you use, which pipeline stages matter, what "good" looks like for prioritization, optional historical close rates for forecasting.
- **Brand context** — brand colors, typography, tone-of-voice rules, optional path to your brand guide.
- **Owner info** — anything not already in shared identity.

Saved to `references/user-context.md` (gitignored).

You can re-run `/setup-core` anytime to update.

## Companion plugins

This plugin is a hub — many other plugins delegate to its subagents:

- **weekly-outreach + plan-tomorrow** call `pipeline-analyst` for weekly/daily prioritization.
- **time-tracking + client-status** integrate with `pipeline-analyst` to surface revenue-vs-time and engagement-vs-pipeline views.
- All plugins benefit from `/diagnose` to verify setup state.
- `/register-schedules` orchestrates standing schedules across the whole marketplace.

## What's inside (file tree)

```
.claude-plugin/plugin.json     Plugin manifest
agents/
  pipeline-analyst.md          Subagent: weekly CRM pipeline ranking
  pipeline-forecast.md         Subagent: forward-looking revenue projection
commands/
  setup-core.md                Interview and config writer
  review-deliverable.md        Slash command for deliverable QA
  diagnose.md                  Ecosystem health check
  log-agent-run.md             Telemetry: append to agent-log.jsonl
  agent-metrics.md             Telemetry: read-only log digest
  register-schedules.md        Bulk-register schedules from library
skills/
  setup/SKILL.md               Auto-fires on setup phrases
  review-deliverable/SKILL.md  Auto-fires on review phrases
  diagnose/SKILL.md            Auto-fires on diagnose phrases
  log-agent-run/SKILL.md       Auto-fires on log phrases
  agent-metrics/SKILL.md       Auto-fires on metrics phrases
  register-schedules/SKILL.md  Auto-fires on schedule phrases
references/
  user-context.template.md     Structure (committed)
  user-context.md              Your config (gitignored, created by setup)
  agent-log-schema.md          JSONL schema for the agent-run log
  schedules.template.md        Starter schedule library (user-editable)
  schedules.md                 Your active schedules (gitignored after first /register-schedules)
```

## Dependencies

- **CRM connector** (HubSpot / Salesforce / Pipedrive / Attio / etc.) — Pipeline Analyst and Pipeline Forecast infer structure from properties.
- **File access** for Deliverable Reviewer — Drive connector or local path.
- **Cowork's scheduled-tasks tool** — required for `/register-schedules` to register schedules.

## License

MIT. See `LICENSE`.
