# Security Policy

## What this plugin does with your data

Core Ops is the shared toolkit plugin: hosts `pipeline-analyst` and `pipeline-forecast` subagents (CRM intelligence), provides `/review-deliverable` (file QA), `/diagnose` (ecosystem health), telemetry (`/log-agent-run`, `/agent-metrics`), and the schedule library (`/register-schedules`).

**Reads:**
- **CRM** — for `pipeline-analyst` and `pipeline-forecast` subagents (deals, contacts, pipeline stages, owners, custom properties).
- **Files** — for `/review-deliverable`, reads the deliverable file you point at (local path or Drive URL).
- **Plugin references** — `references/user-context.md` (CRM details, brand context), `references/schedules.md` (schedule library), `references/agent-log-schema.md` (documentation).
- **Agent log** — `~/.brightway-state/agent-log.jsonl` (read-only by `/agent-metrics`).
- **Shared user-level config** — `~/Documents/Claude/identity.md` (read-only).
- **Cortex memory** (if installed) — for `/diagnose` to verify cortex initialization status. Read-only.

**Writes:**
- **Plugin user-context** — `references/user-context.md` (after `/setup-core`).
- **Plugin schedules.md** — `references/schedules.md` (annotated with `last_registered_id` after `/register-schedules`).
- **Agent log** — `~/.brightway-state/agent-log.jsonl` (append-only by `/log-agent-run`).
- **Cowork scheduled tasks** — `/register-schedules` registers entries with Cowork's scheduled-tasks system, with explicit user confirmation per registration.

**Does not:**
- **Modify the CRM.** Pipeline subagents are read-only.
- **Modify deliverables.** `/review-deliverable` produces findings; never edits the file.
- **Auto-register schedules.** Always shows the list and waits for "y" before any registration.
- **Modify the agent log.** Append-only by design; `/agent-metrics` is strictly read-only.
- **Log message content or sensitive data** — the agent-log captures only meta-records (agent / parent skill / confidence / user action). See `references/agent-log-schema.md` for the explicit privacy guidance.
- **Run scheduled tasks.** Only registers them; execution is owned by Cowork's scheduled-tasks system.

## Where data lives

- Plugin reference files inside the installed plugin directory.
- Agent log at `~/.brightway-state/agent-log.jsonl` (your machine only; never sent off-device).
- Shared identity (read-only) at `~/Documents/Claude/identity.md`.

## What gets sent off your machine

- Whatever your authorized CRM connector sends when `pipeline-analyst` / `pipeline-forecast` invoke it.
- For `/review-deliverable` reading from Drive: whatever your Drive connector sends.
- The agent log never leaves your machine — it's a local observability file.

## Supported versions

| Version | Supported |
|---------|-----------|
| 0.1.x   | Yes       |

## Reporting a vulnerability

Report privately via GitHub Security Advisories:

https://github.com/BrightWayAI/core-ops/security/advisories/new

Do not open a public issue for security concerns. We aim to respond within 5 business days.
