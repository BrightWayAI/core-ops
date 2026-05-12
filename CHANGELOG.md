# Changelog

All notable changes to core-ops are documented here.

Format follows [Keep a Changelog](https://keepachangelog.com/). Versions match `plugin.json`.

## [0.3.0] — Nucleus visibility layer (2026-05-12)

### Added
- **`/nucleus-status`** — at-a-glance text snapshot of the Nucleus stack. Installed plugins, setup state (configured/template/missing), connector availability, recent activity, memory health, anything broken. Terse output (~30-40 lines), fits in chat, scannable in 10 seconds. Read-only. Zero model calls.
- **`/nucleus-dashboard`** v1 — rich visual dashboard as a Cowork HTML artifact titled "Nucleus Dashboard." Six section cards: Stack overview, This week (activity histogram + top skills + rusty skills), Cortex memory health (decay state distribution + person pages + triage log), Outreach pipeline (lead-engine), Time + invoicing (time-tracking), Impact loop (v1 placeholder; v1.1 ships content). Re-renders on demand via `/nucleus-dashboard`. Markdown snapshot fallback at `<config-root>/nucleus-dashboard.md` in Claude Code. Zero model calls.
- **`references/nucleus-dashboard-template.html`** — bundled HTML template with inline SVG charts. No external libraries.
- **Both commands gracefully degrade.** Missing connectors, missing memory, missing telemetry — each section renders an honest placeholder rather than failing the whole render.

### Why this matters
With the marketplace at 13 plugins and the second-brain layer in active use, users (starting with the solo founder, scaling to teams) need a way to see what's installed, what's configured, what's running, and what their system is producing — without grepping the filesystem manually. `/nucleus-status` is the quick check; `/nucleus-dashboard` is the live surface. Both sit alongside `/diagnose` (troubleshooting) in core-ops as the visibility cluster.

### Roadmap
- v1.1 adds the Impact loop section content (drafted→sent ratios across drafting plugins, mining proposal accept rate, P0 completion rate). Deferred from v1 because the metrics need a week of telemetry to populate sensibly.

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
