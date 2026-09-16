---
description: Audit the user's plugin ecosystem health. Checks shared config files (identity, voice, cortex memory), prompts the user about plugin installs, verifies subagent availability, and produces a green/red checklist with specific fix instructions. Run this when something feels off, when onboarding to the marketplace, or when a plugin isn't behaving as expected.
---

# /diagnose

Health check for your marketplace plugin ecosystem. Reports what's set up correctly, what's missing, and what to do about it.

This command is part of `ops` because the toolkit plugin is the natural place for cross-cutting diagnostics. You can invoke it whenever something feels broken or whenever you've added/changed plugins and want to confirm everything's wired up.

---

## Step 1 — Check shared config files

These are the canonical files every plugin reads. Check each:

### 1A — Shared identity (`<config-root>/memory/me/identity.md`)
- **File exists?**
- **Has all required sections** (Person, Company, Primary tools, Communication defaults)?
- **No placeholder values** (e.g., no `[your name]` left in)?

If missing or incomplete: ✗ → "Run `/setup-identity` (cortex) to capture name/company/role/tools once. Every plugin reads it."

### 1B — Shared voice (`<config-root>/memory/me/voice.md`)
- **File exists?**
- **Has voice descriptors, banned phrases, sign-off, hook patterns**?

If missing: ✗ → "Run `/setup-voice` (comms) to capture writing voice once. Every drafting plugin reads it."

### 1C — Cortex memory (`<config-root>/memory/DASHBOARD.md`)
- **File exists?**
- **Has at least one node**?
- **`memory/me/user.md` exists**?

If missing: ✗ → "Cortex hasn't been initialized. Either install cortex or, if installed, restart Cowork to trigger initialization."

### 1D — Memory-as-git (`<config-root>/memory/.git/`)

Versioned memory powers `/morning`'s overnight-diff review and rollback safety (cortex v4.12.0+). Report a one-line health status (this is informational — memory-as-git is optional):

- **Enabled?** Does `<config-root>/memory/.git/` exist?
  - **No** → ⚠ (not an error) → "Memory-as-git not initialized — diffs/rollback unavailable. Run `/setup-identity` (cortex) to enable, or set `memory_as_git.enabled: true` in `cortex.user-context.md`."
  - **Yes** → continue.
- **Last commit when?** `git -C <config-root>/memory log -1 --format='%cd (%s)' --date=short`. If the most recent commit is older than ~3 days on a working day, note: "Last memory commit was <date> — `/end-day` Step 5.8 commits daily; run `/end-day` to catch up."
- **Working tree clean?** `git -C <config-root>/memory status --porcelain | wc -l`. If non-zero, note: "<N> uncommitted memory changes — next `/end-day` will commit them, or commit manually."
- **Remote configured?** Read `memory_as_git.remote` from `cortex.user-context.md` (and/or `git -C <config-root>/memory remote -v`). If set, report the remote + whether `push_on_close` is on; if a remote is set but the local is ahead of it, note "N commits not pushed." If no remote, report "local-only (default)" — not a problem, just informational.

Surface all of the above as a single `Memory-as-git: <enabled/disabled> · last commit <date> · <clean/N dirty> · <local-only | remote: pushed/N behind>` line in the Step 5 checklist.

### 1E — Memory cap violations (v4.16+, Nucleus Operating Model Refactor Phase 2 step 2.3)

```
python3 scripts/cortex_cli.py check-caps --memory-root <config-root>/memory
```

(This is a cortex script — resolve its path relative to the cortex plugin, same pattern as any other cross-plugin script call.) Read-only, deterministic — same check `/reindex` and `/cleanup` Section M run. Report as a single line: `Memory caps: clean` or `Memory caps: <N> FAIL, <M> WARN — run /cleanup for detail`. A FAIL here means `/recall`'s default load boundary (< 4K tokens per node) is broken for at least one node — treat as a ✗ in Step 5, not just informational.

### 1F — Scheduled-loop health

Scheduling is optional, but a configured loop must be observable:

- Definitions: `<config-root>/plugins/ops/schedules.md`
- This host's registration record:
  `<config-root>/plugins/ops/schedule-registrations/<host-id>.json`
- Receipts: `<config-root>/plugins/ops/schedule-runs/<schedule>/`

If the scheduler can list tasks, reconcile live state; live state wins over cached
IDs. For `nightly-listen`, report one of:

- `not configured` — informational, not an error;
- `defined, not registered` — warning with `/register-schedules`;
- `registered, never observed` — warning until the first scheduled window passes;
- `healthy` — most recent success/partial receipt is within 36 hours;
- `stale/failed` — red when two expected windows passed without a receipt, or the
  latest receipt failed. Show sanitized error codes and the host scheduler history
  location; never print connector payloads.

A registered ID alone is never green evidence that the agent loop runs.

---

## Step 2 — Check plugin setup state

For each marketplace plugin, file paths depend on Cowork's plugin install location, which varies. Instead of probing the filesystem, ask the user.

Run this dialog:

> "I'm going to ask you which plugins you have installed. For each one, I'll verify it's been set up correctly."

Then for each plugin in the marketplace catalog:

| Plugin | Setup command | What to verify |
|---|---|---|
| cortex | (foundation) | `/recall` returns useful, bounded context |
| ops | `/setup-core` | pipeline analysis reads configured CRM stages |
| briefing | `/setup-brief` | `/brief` lists unavailable sources honestly |
| growth | `/setup-relationships` | `/relationships` builds or cleanly empties its queue |
| clients | `/setup-projects`, `/setup-status` | `/project-setup` uses a real offering, `/client-status` drafts only, and `<config-root>/plugins/clients.sow-template.md` exists (else `/sow` note: "SOW template not configured — outputs use generic default styling; run `/setup-projects` with a sample SOW to fix") |
| admin | `/setup-time` | `/track-time` can classify pasted or calendar events |
| comms | `/setup-style` | `/style` reads the canonical voice file |
| research | `/setup-news` | `/roundup` doesn't error on its source gate |
| alignment | `/setup` | `/scan` names a missing Slack source instead of inventing activity |

For each plugin the user says they have:
- "Have you run the setup command?" (Y/N)
- If N → ✗ "Run `[setup-command]` before relying on this plugin."
- If Y → ✓ "Looks set."

Don't try to programmatically open plugin user-context files — paths vary too much. Trust the user's report and surface the verification step they should run themselves.

---

## Step 3 — Check subagent availability

Each subagent is registered in Claude's `subagent_type` enum when its plugin is installed. Walk through the canonical subagents:

| Subagent | Lives in | Quick test |
|---|---|---|
| `memory-librarian` | cortex | "Try `/search` with a broad query — does it route to memory-librarian?" |
| `note-taker` | cortex | "Used by `/listen`; unavailable source modes are disclosed." |
| `relationships-director` | growth | "Used for ranking and single-contact research." |
| `pipeline-analyst` | growth | "Available to growth, briefing, clients, and pipeline reviews." |
| `pipeline-forecast` | growth | "Used for monthly or explicit forecasts." |
| `news-curator` (agent) | research | "Used by `/roundup`." |
| `post-assembler` | comms | "Used by `/post` (drafts staged `/roundup` candidates)." |
| `alignment-scanner` | alignment | "Used by Slack scan, pulse, report, and risk updates." |

For each: report whether the parent plugin is installed (per Step 2). If yes → ✓. If no → ✗ "Install [plugin] to make this subagent available."

If the user reports a subagent isn't being invoked when expected, suggest:
1. Restart Cowork (subagent enum sometimes requires reload)
2. Verify the plugin is installed and the agent file is in `[plugin]/agents/`
3. Check the Task tool's `subagent_type` enum — the agent name should appear there

---

## Step 4 — Check connector availability

Connectors (CRM, email, calendar, etc.) are installed via Cowork's connections panel separately from plugins. Per identity.md's "Primary tools" section, ask the user to confirm each is connected:

For each tool listed in `<config-root>/memory/me/identity.md` Primary tools section:
- "[Tool name] connected in Cowork? (Y/N)"
- If N → ✗ "Plugins that need [tool] will fall back to inline mode or fail. Connect via Cowork → Connections → [tool]."
- If Y → ✓

Common connectors:
- HubSpot / Salesforce / Pipedrive (CRM)
- Gmail / Outlook (email)
- Google Calendar / Outlook Calendar
- Slack
- Google Drive
- Apollo (for growth enrichment, when configured)
- Granola or another note source (for note-taker transcript mode)

---

## Step 5 — Output the checklist

Render a clear summary:

```
## Plugin Ecosystem Health Check — [today's date]

### ✓ Good
- [bulleted list of green items]

### ✗ Needs attention
- [red items, each with the specific fix command]

### Recommended next action
[The single highest-impact thing to fix first. Examples:
"Run /setup-identity to capture the canonical identity once — unblocks 7 other plugins."
"Restart Cowork — your subagents aren't being recognized."
"Connect Apollo in Cowork connections — lead-pull won't work without it."]

### Want me to walk through any of the fixes?
```

Then offer to step through the highest-priority fixes interactively.

---

## Behavior rules

- **Don't auto-fix.** This is diagnosis, not surgery. Surface what's broken and recommend the fix, but let the user run the fix.
- **Ask, don't assume.** Plugin install state and connector state are user-dependent — ask rather than guess.
- **Lead with the highest-impact red item.** Don't dump 15 issues equally; rank by what unblocks the most.
- **Be specific.** "Identity is missing" is unhelpful. "Identity file at `<config-root>/memory/me/identity.md` is missing — run `/setup-identity` (in cortex)" is helpful.
- **Don't be alarmist.** A missing optional file (e.g., voice.md) is fine if the user doesn't draft. Note it as informational, not red.

---

## When this is most useful

- **Onboarding** — first-time setup of the marketplace
- **Something feels broken** — agent isn't invoking, connector seems off
- **After a major change** — you installed/removed plugins, restarted Cowork, switched machines
- **Before a busy day** — quick "is everything wired up?" check before the workflows that depend on it

Run `/diagnose` once a month or whenever you suspect something. Takes ~3 minutes.
