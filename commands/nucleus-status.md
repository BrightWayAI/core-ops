---
description: At-a-glance snapshot of your Nucleus stack — installed plugins, setup state, connector availability, last-runs, anything broken. Terse text output in chat, scannable in 10 seconds. Run any time. For a richer visual surface use `/nucleus-dashboard`; for deep troubleshooting use `/diagnose`.
---

# /nucleus-status

One-screen view of your Nucleus stack health. Three sibling commands serve overlapping but distinct needs:

| Command | Purpose | Output |
|---|---|---|
| `/nucleus-status` | "Is everything wired up?" | Terse text in chat, ~10 second read |
| `/nucleus-dashboard` | "What's my system doing this week?" | Rich Cowork artifact with activity + impact metrics |
| `/diagnose` | "Something's broken — what's wrong and how do I fix it?" | Per-issue troubleshooting steps |

This command is the terse check. No diagnostics, no walkthroughs — just a snapshot.

---

## Step 0 — Resolve config root

Ensure access to `~/Documents`. In Cowork, call `request_cowork_directory(~/Documents)` once if not already granted. In Claude Code (or any environment with direct filesystem access), no mount is needed. Then read `~/Documents/.claude-plugin-config-root`.

- **Pointer missing** → stop with: "No plugin config root configured. Run any plugin's `/setup-*` command to bootstrap, then re-run `/nucleus-status`."
- **Pointer exists** → read line 1 → that's `<config-root>`. Ensure access to it. Continue.

---

## Step 1 — Gather state (read-only, in parallel where possible)

### A — Shared config

For each shared config file, capture: exists? populated? (size > 0 and contains expected sections)

| File | Path | Owned by |
|---|---|---|
| Identity | `<config-root>/identity.md` | cortex `/setup-identity` |
| Voice | `<config-root>/voice.md` | cortex `/setup-voice` |
| Cortex DASHBOARD | `<config-root>/memory/DASHBOARD.md` | cortex auto-commit |
| Decay config (v4.4+) | `<config-root>/memory/.decay-config.md` | cortex (auto-created on first /recall) |
| Note sources (v4.3+) | `<config-root>/plugins/cortex.note-sources.md` | cortex `/setup-sources` |
| Scope migration marker | `<config-root>/memory/.scope-migration-done` | cortex `/end-day` (one-time) |

### B — Installed plugins (per-plugin setup state)

List the known plugin set from the marketplace catalog. For each, check whether its `user-context.md` exists at the expected path in `<config-root>/plugins/`. The set:

```
claude-cortex        → identity.md + voice.md (no per-plugin context file)
core-ops             → core-ops.user-context.md
lead-engine          → lead-engine.user-context.md
bizdev-outreach      → bizdev-outreach.user-context.md
weekly-outreach      → weekly-outreach.user-context.md
referral-engine      → referral-engine.user-context.md
news-curator         → news-curator.user-context.md
daily-brief          → daily-brief.user-context.md
plan-tomorrow        → plan-tomorrow.user-context.md
project-setup        → project-setup.user-context.md
time-tracking        → time-tracking.user-context.md
client-status        → client-status.user-context.md
weekly-alignment     → weekly-alignment.org-context.md
writing-style        → writing-style.user-context.md
```

Per plugin, the state is one of:
- **configured** — user-context file exists, populated, not template-default
- **template** — file exists but still contains template placeholders (e.g., `[your value here]`)
- **missing** — file doesn't exist (plugin likely not installed or not yet set up)

Cap detection at file existence + a one-line check for obvious template strings. Don't do deep validation — that's `/diagnose`'s job.

### C — Connectors (best-effort detection)

Try to detect which MCP connectors are available in the current runtime. Each detection is a probe call to a low-cost read endpoint:

| Connector | Probe |
|---|---|
| HubSpot | `mcp__*hubspot*__get_user_details` or similar (any HubSpot tool exists) |
| Gmail | `mcp__*gmail*__list_labels` |
| Calendar | `mcp__*calendar*__list_calendars` |
| Drive | `mcp__*drive*__list_recent_files` |
| Granola | `mcp__*granola*__list_meetings` |
| Slack | `mcp__*slack*__channels_list` or `mcp__*slack*__usergroups_me` |
| Session info | `mcp__*session*__list_sessions` |
| Cowork artifacts | presence of `mcp__cowork__create_artifact` |

Probe = "is the tool name available in this runtime?" Don't actually call the tools — just check whether the runtime exposes them. The MCP tool list is discoverable without execution.

Each connector status: **connected** | **not detected**.

### D — Last activity (if core-ops agent-log exists)

Read `~/.brightway-state/agent-log.jsonl` if it exists. Last 7 days. Aggregate:

- **Recent commands run** — count by skill name, top 5 sorted desc
- **Last run timestamp per skill** — most recent invocation

If the log doesn't exist, skip this section silently (telemetry is optional).

### E — Memory health (v4.4+)

If cortex memory directory exists, do a quick scan:

- **Node count** — number of `.md` files in `<config-root>/memory/` (excluding `archive/`, `person/archive/`, hidden files, `DASHBOARD.md`, `CLAUDE.md`)
- **Person pages active vs. archived** — count files in `memory/person/` vs. `memory/person/archive/`
- **Knowledge entry state distribution** — sample-scan up to 20 random entries across nodes; classify each as Fresh / Stale / Dormant / Cold per the decay model. Report percentages.

This is best-effort — don't read every entry; sample.

### F — Brief snapshot (if daily-brief installed)

Check `<config-root>/briefs/` directory:
- Most recent brief file name and date
- Count of brief files in the last 30 days (rough proxy for daily-brief usage)

---

## Step 2 — Render the status block

Format the output as a single compact block. Aim for ~30-40 lines of terminal-readable text. Sections are tab-aligned. Use ✓ / ✗ / ⚠ markers for state. Don't include sections that have nothing to report.

```
NUCLEUS STATUS · <today's date>
Config root: <config-root>

SHARED CONFIG
  ✓ identity.md          populated
  ✓ voice.md             populated
  ✓ memory/DASHBOARD.md  <N> active nodes
  ✓ .decay-config.md     defaults in effect
  ✓ note-sources.md      <N> sources enabled
  ⚠ .scope-migration-done  missing — first /end-day will run the migration

PLUGINS (14)
  ✓ configured  (10): claude-cortex, core-ops, lead-engine, bizdev-outreach,
                       weekly-outreach, referral-engine, news-curator,
                       daily-brief, plan-tomorrow, project-setup
  ⚠ template   (1):  writing-style (run /setup-style to fill in)
  ✗ missing    (3):  time-tracking, client-status, weekly-alignment

CONNECTORS
  ✓ HubSpot      ✓ Gmail         ✓ Calendar      ✓ Drive
  ✓ Granola      ⚠ Slack (not detected)         ✓ Cowork artifacts

RECENT ACTIVITY (last 7 days)
  /end-day           3 runs   last: yesterday
  /brief             3 runs   last: yesterday
  /process-brief     2 runs   last: 2d ago
  /weekly-outreach   1 run    last: 4d ago
  /referral-ask      1 run    last: 5d ago

MEMORY HEALTH (v4.4)
  Active nodes:        12
  Person pages:        7 active, 0 archived
  Entry distribution:  Fresh 60% · Stale 25% · Dormant 12% · Cold 3%
  Last /rehearse:      5d ago

DAILY BRIEF
  Last brief:    yesterday (2026-05-11)
  Last 30 days:  18 briefs generated

NOTHING BROKEN
```

If anything is broken (missing pointer file, plugin in broken state, expected file unreadable), add a `BROKEN` section at the end with one-line fixes per item. Otherwise emit `NOTHING BROKEN` as the closing line.

---

## Step 3 — Optional follow-up

If everything is green, exit cleanly. If anything is in `template` or `missing` state, finish with a one-line nudge:

> "Some plugins not set up. Run `/diagnose` for per-plugin walkthrough, or `/nucleus-dashboard` for the visual surface."

Don't loop back into other commands automatically. The user decides.

---

## Behavior rules

- **Read-only.** No file writes, no config changes, no telemetry log (this command is itself excluded from `/log-agent-run` to avoid recursion noise).
- **Terse.** ~30-40 lines of output. Anything longer belongs in `/nucleus-dashboard`.
- **Cap probes.** No deep file reads. No connector tool calls. State detection is filesystem + tool-name existence only.
- **Fast.** Target < 2 seconds wall time. The user runs this when they want a quick check, not a deep audit.
- **Honest about ambiguity.** If a plugin's user-context file exists but contains unparseable garbage, mark it `⚠` rather than `✓` or `✗`. Don't crash on bad input.

## What this command is NOT for

- **Troubleshooting** — that's `/diagnose`. Status surfaces what's broken; diagnose explains why and how to fix.
- **Visual dashboard / weekly review** — that's `/nucleus-dashboard`.
- **Per-plugin deep audit** — that's the plugin's own `/setup-*` command run with the "update" path.
- **Memory cleanup** — that's `/cleanup`.
- **Telemetry analysis** — that's `/agent-metrics`.
