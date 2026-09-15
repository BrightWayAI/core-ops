# Changelog

All notable changes to ops (formerly core-ops) are documented here.

Format follows [Keep a Changelog](https://keepachangelog.com/). Versions match `plugin.json`.

## [0.10.0] — `/setup-voice` ownership moves to Comms Desk (2026-09-15)

### Changed
- `README.md`, `commands/status.md`, `commands/diagnose.md` updated to attribute `/setup-voice` to the `comms` plugin (Comms Desk) instead of `cortex` — the interview moved there, while `<config-root>/memory/me/voice.md` remains a Cortex-owned canonical file.

## [0.8.0] — pipeline agents move to growth (2026-09-15)

### Changed
- `pipeline-analyst` and `pipeline-forecast` subagents (and their Codex mirrors) moved to the `growth` plugin. `chief-of-staff` (`/cos`) now delegates to `growth:pipeline-analyst` / `growth:pipeline-forecast` when growth is installed, and reports "Growth Engine not installed; pipeline analysis skipped" otherwise. Ops no longer reads the CRM directly.
- Updated README, SECURITY.md, `/diagnose`, `/setup-core`, `skills/setup/SKILL.md`, and reference templates to reflect the new ownership.

## [0.9.0] — `/dashboard` moved to briefing (2026-09-15)

### Removed
- `/dashboard` (rich visual Cowork-artifact stack surface), its skill, and `references/nucleus-dashboard-template.html` moved to the `briefing` plugin (Today's Brief), per the "Today's Brief is every what's-going-on surface at any cadence" container rule. `ops` retains `/status` (terse text snapshot) and `/diagnose` (troubleshooting) — these explicitly did NOT move.

### Changed
- `/nucleus-dashboard` (the existing deprecated alias) now routes to `briefing:dashboard` instead of an in-plugin `/dashboard`, with a degraded-mode skip + `/status` suggestion if `briefing` isn't installed.
- `/status` and its skill updated to point at briefing's `/dashboard` for the richer visual surface.

## [0.7.0] — Renamed to ops (2026-09-15)

### Changed
- Renamed from `core-ops` to `ops` (display name: Chief of Staff) as part of the 2026-09-15 Nucleus plugin rename. Old plugin ID/repo name redirects; see marketplace catalog.
- `/nucleus-status` renamed to `/status`; `/nucleus-dashboard` renamed to `/dashboard`. Old command names remain as deprecated one-line aliases.

## [0.6.4] — routing and schedule-state hardening (2026-09-15)

### Changed
- Split chief-of-staff route planning from parent-owned execution with a structured route-plan contract and narrower triggers.
- Moved schedule definitions, per-host IDs, and metadata-only run receipts into separate user-owned config-root paths.
- Updated status, diagnostics, dashboard, and connector ownership for the nine-plugin catalog.

## [0.6.3] — Codex adapter synchronization (2026-09-15)

### Added
- Read-only Codex binding and OpenAI host preamble for the `chief-of-staff` `/cos` entrypoint.

### Fixed
- Updated Codex metadata and degraded behavior after deliverable QA moved to `delivery`.

## [0.6.2] — claude plugin eval suite (2026-09-15)

Nucleus Operating Model Refactor Phase 4 step 4.2.

### Added
- `evals/chief-of-staff-natural-language/` — eval case testing the chief-of-staff agent's open-ended natural-language routing, with an `llm` grader
  checking the response acts on the natural-language request directly rather
  than asking the user to type the explicit command. Run with
  `claude plugin eval . --case chief-of-staff-natural-language`; `--ablation with-without`
  (the default when the plugin resolves) reports the delta between installed
  and not — a delta near zero means the skill's description isn't matching
  natural phrasing and needs work.
- `.gitignore` — excludes `evals/results/` (per-run output, not checked in).

## [0.6.1] — Skill auto-invocation audit (2026-09-15)

Nucleus Operating Model Refactor Phase 3 step 3.7. Ritual and side-effecting
skills marked `disable-model-invocation: true` so they only run on explicit
invocation, not loose natural-language matching — the model can still be
asked to run them by name. Read-mostly, low-stakes, or high-frequency
conversational skills are left auto-invocable. Marketplace-wide this brings
model-invocable skills from ~81 to 27, under the ≤30 target audited with
`/skill-doctor`.

### Changed
- Marked `disable-model-invocation: true` on: `log-agent-run`, `register-schedules`, `setup`, `setup-core`, `test-connectors`.

## [0.6.0] — chief-of-staff agent + /cos, nucleus-router retired (2026-09-15)

Nucleus Operating Model Refactor Phase 3 step 3.1. Adds the natural-language
front door that replaces the standalone `nucleus-router` plugin.

### Added
- `agents/chief-of-staff.md` — job-description-style agent (purpose, goals,
  success/failure criteria, escalation rules, tools). Loads
  `<config-root>/memory/hot.md` + `index.md`, reads the autonomy policy in
  `memory/CLAUDE.md`, and routes natural-language or role-addressed requests
  to the matching installed plugin command. Narrates read-only/drafting work;
  confirms before ASK FIRST/NEVER-tier actions.
- `/cos` command — explicit invocation of the agent.
- `skills/cos/SKILL.md` — always-loaded routing skill, thin wrapper around
  the agent (no inline intent tables — routing logic lives in the job
  description, per the refactor's "agents are job descriptions, not
  procedures" principle).

### Removed
- Retired the `nucleus-router` plugin entirely (archived on GitHub with a
  redirect README pointing at `/cos` in this plugin; local clone removed).
  Cross-references in `relationships` skill docs and cortex's
  `references/autonomy.md` updated to point at the chief-of-staff agent.

## [0.5.0] — /review-deliverable moved to delivery (2026-09-15)

Nucleus Operating Model Refactor Phase 3 step 3.3. `project-setup` renamed to
`delivery` and absorbed `client-status` + this plugin's `/review-deliverable`.

### Removed
- `commands/review-deliverable.md`, `skills/review-deliverable/SKILL.md` —
  moved to the `delivery` plugin. Brand/CRM config (`core-ops.user-context.md`)
  stays owned by this plugin; `delivery` reads it cross-plugin, same pattern
  as `relationships` reading CRM config from here.

## [0.4.4] — writing-style renamed to voice (2026-09-15)

### Changed
- `commands/nucleus-status.md` — plugin-name and config-file references updated
  from `writing-style` to `voice` (Nucleus Operating Model Refactor Phase 3
  step 3.4).

## [0.4.3] — Identity/voice moved to memory/me/ (2026-09-15)

### Changed
- Path references updated from `<config-root>/identity.md` / `<config-root>/voice.md` to `<config-root>/memory/me/identity.md` / `<config-root>/memory/me/voice.md`, per the Nucleus Operating Model Refactor Phase 2 scopes restructure (identity/voice are personal, not org-shared facts). No behavior change beyond the path.

## [0.4.2] — Diagnose reports memory cap violations (2026-09-15)

### Added
- `/diagnose` Step 1E — reports cortex's `check-caps` output as a health line
  (`Memory caps: clean` or `Memory caps: N FAIL, M WARN`). A FAIL is treated
  as a ✗, since it means `/recall`'s default load boundary is broken for at
  least one node. Part of the Nucleus Operating Model Refactor, Phase 2 step
  2.3.

## [0.4.1] — Instantiate schedule library (2026-09-15)

### Added
- `references/schedules.md`, instantiated from the template for the first time (zero
  schedules had ever been registered). Populates `nightly-listen` only, with a
  run-window guard and the exact registration prompt for `/register-schedules` to hand
  to Cowork. Part of the Nucleus Operating Model Refactor, Phase 1 step 1.4.

## [0.4.0] — Live connector certification (2026-09-15)

### Added
- `/test-connectors` and its matching skill. The workflow performs real, bounded,
  read-only calls against authorized calendar, mail, CRM, Slack, Drive, contact
  enrichment, and transcript connectors.
- Metadata-only JSON evidence compatible with Nucleus's deterministic connector-report
  validator and coordinated release gate.
- Explicit cross-host certification: Claude and ChatGPT runs remain separate because
  connector authorization and tool schemas are host-specific.

## [0.3.3] — OpenAI host adapter (2026-09-14)

### Added
- Native Codex/ChatGPT plugin manifest, durable `AGENTS.md` entrypoint, and an explicit OpenAI capability/degradation contract.
- GPT-discoverable skill aliases for canonical command workflows and read-only Codex role bindings where this plugin ships agents.
- Shared config-root resolution compatible with Cortex and Claude; all GPT tests use repository fixtures or temporary directories only.

## [0.3.2] — /diagnose memory-as-git health line (2026-06-08)

### Added
- `/diagnose` Step 1D — memory-as-git health check (closes the last build-plan item of the cortex memory-as-git proposal). Reports enabled? / last-commit / clean-tree / remote status as one informational line in the Step 5 checklist. Coordinated with cortex v4.13.0.

## [0.3.1] — Schedule library: /listen + /research-gaps (2026-05-16)

### Added
- `nightly-listen` schedule row in `references/schedules.template.md` — registers `/listen` (cortex v4.7+) at 11pm daily for unattended overnight ingest. Recommended for users running cortex with `/setup-sources` configured.
- `weekly-research-gaps` schedule row — registers `/research-gaps` (cortex v4.5+) at 9am Saturday for weekly autonomous gap-fill scans.

### Why
cortex v4.7 shipped `/listen` for nightly autonomous ingest. Without a registered cron, the pipeline never runs. This patch adds the schedule entry so `/register-schedules` automatically wires it. Same for `/research-gaps`'s recommended weekly cadence.

### Migration
Existing users on v0.3.0 should re-run `/register-schedules` after updating to v0.3.1. The new rows are registered as new entries; pre-existing schedules remain (idempotent — register-schedules uses the `last_registered_id` annotation to skip duplicates).

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
