# Changelog

All notable changes to core-ops are documented here.

Format follows [Keep a Changelog](https://keepachangelog.com/). Versions match `plugin.json`.

## [Unreleased]

### Added
- `pipeline-forecast` subagent — forward-looking revenue projection. Companion to `pipeline-analyst`. Uses stage probabilities × deal values × historical close rates to project a future window. Returns scenarios (best / expected / worst / stretch), needle-mover deals, gap-to-target analysis, structural risks. Useful for monthly planning, runway analysis, board reporting.
- `/diagnose` — ecosystem health check. Audits shared config files (`identity.md`, `voice.md`, cortex memory), plugin setup state, subagent availability, connector wiring. Surfaces specific fix instructions ranked by impact.
- `/log-agent-run` and `/agent-metrics` — lightweight telemetry. Append-only meta-log at `~/.brightway-state/agent-log.jsonl` capturing agent / parent skill / confidence / user action. Privacy-respecting (no message content, no sensitive data). `/agent-metrics` produces read-only digests for trend analysis.
- `/register-schedules` and `references/schedules.template.md` — versioned schedule library. Bulk-register standing schedules with Cowork's scheduled-tasks system. Useful for new-machine setup and reproducibility across machines.
- `references/agent-log-schema.md` — JSONL schema documentation for the agent-run log.

## [0.1.0] — Initial release

### Added
- Plugin renamed from `brightway-core` to `core-ops` for generic reuse (BrightWay-specific naming removed from plugin name).
- `pipeline-analyst` subagent — point-in-time CRM ranking. Scores and ranks pipeline by recency × signal strength × stage. Used by `weekly-outreach`, `plan-tomorrow`, any pipeline-review workflow.
- `/review-deliverable` slash command — structured QA pass on a client deliverable (deck, doc, spreadsheet, one-pager) against the user's brand guide and the original brief. Returns location-tagged findings ranked by severity, plus a ship/no-ship verdict.
- `/setup-core` interview — captures CRM context, brand context, owner info. Writes to `references/user-context.md` (gitignored).
- Generic by design — works with any CRM and any brand once configured.
