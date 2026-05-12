# Changelog

All notable changes to core-ops are documented here.

Format follows [Keep a Changelog](https://keepachangelog.com/). Versions match `plugin.json`.

## [0.2.3] — Platform-agnostic Step 0 (2026-05-12)

### Changed
- **Setup command Step 0 now platform-agnostic.** Every `request_cowork_directory(...)` call is conditional: "In Cowork, call `request_cowork_directory(...)`. In Claude Code (or any environment with direct filesystem access), no mount is needed." Same plugin source works in both runtimes.

### Why this matters
Phase 0 of SECOND-BRAIN-V2-SPEC. Removes the implicit Cowork-only assumption so Claude Code users do not hit unsupported tool calls during setup.

## [0.2.0] — Config-root refactor + previously in-flight features

### Changed (config-root refactor)
- **Plugin config moved to a user-chosen folder.** All reads and writes of `references/user-context.md` (which was inside the plugin's source folder — read-only under Cowork's mount) now go to `<config-root>/plugins/core-ops.user-context.md`, where `<config-root>` is the folder the user chooses on first plugin setup (recorded at `~/Documents/.claude-plugin-config-root`). Without this, the plugin's setup writes failed silently under Cowork.
- **`/setup-core` Step 0 now bootstraps the config root.** Reads `~/Documents/.claude-plugin-config-root` first. If missing, prompts the user for the config root, persists their choice, mounts it via `request_cowork_directory`, and offers to migrate existing `~/Documents/Claude/identity.md` / `voice.md` and any pre-staged `~/Documents/Claude/plugin-configs/*.user-context.md` files.
- **All operating skills, commands, and agents** (`/review-deliverable`, `/diagnose`, `/log-agent-run`, `/agent-metrics`, `/register-schedules`, `pipeline-analyst`, `pipeline-forecast`) now read from `<config-root>/plugins/core-ops.user-context.md` instead of the old plugin-relative path.
- **User-facing prompts and skill descriptions debranded.** "BrightWayAI marketplace plugins" → "marketplace plugins"; "BrightWayAI plugin" → "marketplace plugin." The marketplace is fork-friendly — any user can install and run.

### Added (previously in-flight, now formally released)
- `pipeline-forecast` subagent — forward-looking revenue projection. Companion to `pipeline-analyst`. Uses stage probabilities × deal values × historical close rates to project a future window. Returns scenarios (best / expected / worst / stretch), needle-mover deals, gap-to-target analysis, structural risks. Useful for monthly planning, runway analysis, board reporting.
- `/diagnose` — ecosystem health check. Audits shared config files (`identity.md`, `voice.md`, cortex memory), plugin setup state, subagent availability, connector wiring. Surfaces specific fix instructions ranked by impact.
- `/log-agent-run` and `/agent-metrics` — lightweight telemetry. Append-only meta-log capturing agent / parent skill / confidence / user action. Privacy-respecting (no message content, no sensitive data). `/agent-metrics` produces read-only digests for trend analysis.
- `/register-schedules` and `references/schedules.template.md` — versioned schedule library. Bulk-register standing schedules with Cowork's scheduled-tasks system. Useful for new-machine setup and reproducibility across machines.
- `references/agent-log-schema.md` — JSONL schema documentation for the agent-run log.

## [0.1.0] — Initial release

### Added
- Plugin renamed from `brightway-core` to `core-ops` for generic reuse (BrightWay-specific naming removed from plugin name).
- `pipeline-analyst` subagent — point-in-time CRM ranking. Scores and ranks pipeline by recency × signal strength × stage. Used by `weekly-outreach`, `plan-tomorrow`, any pipeline-review workflow.
- `/review-deliverable` slash command — structured QA pass on a client deliverable (deck, doc, spreadsheet, one-pager) against the user's brand guide and the original brief. Returns location-tagged findings ranked by severity, plus a ship/no-ship verdict.
- `/setup-core` interview — captures CRM context, brand context, owner info. Writes to `references/user-context.md` (gitignored).
- Generic by design — works with any CRM and any brand once configured.
