---
name: nucleus-status
description: At-a-glance Nucleus stack snapshot — installed plugins, setup state, connector availability, recent activity, memory health, anything broken. Terse text output in chat, scannable in 10 seconds. Auto-fires on "/nucleus-status", "stack status", "is everything wired up", "show my nucleus", "check my plugins", "what's installed", or any phrase asking for a quick health snapshot. For a richer visual surface use `/nucleus-dashboard`; for troubleshooting use `/diagnose`.
---

See `commands/nucleus-status.md` for the full output format and detection logic.

## When this skill fires

- User runs `/nucleus-status` directly
- User says: "stack status", "is everything wired up", "show my nucleus", "what's installed", "check my plugins", "give me a status check"
- User says they "want a quick check" of their setup

## What this skill is NOT for

- **Troubleshooting** — that's `/diagnose`. Status surfaces problems; diagnose walks you through fixes.
- **Visual dashboard with impact metrics** — that's `/nucleus-dashboard`.
- **Per-plugin configuration** — that's the plugin's own `/setup-*` command.
- **Memory audit** — that's `/cleanup`.

## Inputs

- `~/Documents/.claude-plugin-config-root` pointer file (must exist)
- `<config-root>/identity.md`, `<config-root>/voice.md` (existence + populated check)
- `<config-root>/memory/DASHBOARD.md` (cortex health signal)
- `<config-root>/plugins/*.user-context.md` (per-plugin setup state)
- `<config-root>/memory/.decay-config.md`, `<config-root>/plugins/cortex.note-sources.md`, `<config-root>/memory/.scope-migration-done` (v4.3+/v4.4+ markers)
- Runtime MCP tool list (for connector detection — no actual tool calls made)
- `~/.brightway-state/agent-log.jsonl` (recent activity, optional)
- `<config-root>/briefs/` directory listing (daily-brief usage signal, optional)

## Outputs

- Single status block (~30-40 lines of text) printed to chat
- `NOTHING BROKEN` closing line if all green, or `BROKEN` section with one-line fixes per issue

## Cost

Zero model calls in steady state. Pure filesystem reads + tool-name existence checks. Target < 2 seconds wall time.

## Failure modes

- Pointer file missing → stops with clear message routing to a setup command
- Cortex memory directory missing → memory health section omitted, not an error
- Agent log missing → recent activity section omitted, not an error
- Plugin user-context file unreadable → marks plugin `⚠` and continues; doesn't crash on bad input
