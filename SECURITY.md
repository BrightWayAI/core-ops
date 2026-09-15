# Security Policy

## What this plugin does with your data

Ops (Chief of Staff) is the shared toolkit plugin: hosts the `chief-of-staff` agent (`/cos`), `/diagnose` (ecosystem health), telemetry (`/log-agent-run`, `/agent-metrics`), and the schedule library (`/register-schedules`). As of 2026-09-15, the `pipeline-analyst` and `pipeline-forecast` CRM subagents live in the `growth` plugin; `/cos` delegates to them there when growth is installed, and shows "Growth Engine not installed; pipeline analysis skipped" otherwise.

**Reads:**
- **Plugin references** — immutable templates and schemas in `references/`.
- **Agent log** — `~/.brightway-state/agent-log.jsonl` (read-only by `/agent-metrics`).
- **Shared private profile** — `<config-root>/memory/me/identity.md` (read-only).
- **Cortex memory** (if installed) — for `/diagnose` to verify cortex initialization status. Read-only.

**Writes:**
- **Plugin settings** — `<config-root>/plugins/ops.user-context.md` (after `/setup-core`).
- **Schedule definitions/state/receipts** — `<config-root>/plugins/ops/`; installed references are never mutated.
- **Agent log** — `~/.brightway-state/agent-log.jsonl` (append-only by `/log-agent-run`).
- **Cowork scheduled tasks** — `/register-schedules` registers entries with Cowork's scheduled-tasks system, with explicit user confirmation per registration.

**Does not:**
- **Access the CRM directly.** Pipeline subagents (read-only) now live in the `growth` plugin.
- **Auto-register schedules.** Always shows the list and waits for "y" before any registration.
- **Modify the agent log.** Append-only by design; `/agent-metrics` is strictly read-only.
- **Log message content or sensitive data** — the agent-log captures only meta-records (agent / parent skill / confidence / user action). See `references/agent-log-schema.md` for the explicit privacy guidance.
- **Run scheduled tasks.** Only registers them; execution is owned by Cowork's scheduled-tasks system.

## Where data lives

- Immutable plugin reference files inside the installed plugin directory.
- User settings and schedule state under `<config-root>/plugins/ops/`.
- Agent log at `~/.brightway-state/agent-log.jsonl` (your machine only; never sent off-device).
- Shared identity (read-only) at `<config-root>/memory/me/identity.md`.

## What gets sent off your machine

- Nothing CRM-related — this plugin no longer talks to the CRM directly (see the `growth` plugin's `pipeline-analyst` / `pipeline-forecast` for that).
- The agent log never leaves your machine — it's a local observability file.

## Supported versions

| Version | Supported |
|---------|-----------|
| 0.1.x   | Yes       |

## Reporting a vulnerability

Report privately via GitHub Security Advisories:

https://github.com/BrightWayAI/core-ops/security/advisories/new

Do not open a public issue for security concerns. We aim to respond within 5 business days.
