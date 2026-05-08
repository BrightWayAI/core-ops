---
description: Audit the user's plugin ecosystem health. Checks shared config files (identity, voice, cortex memory), prompts the user about plugin installs, verifies subagent availability, and produces a green/red checklist with specific fix instructions. Run this when something feels off, when onboarding to the marketplace, or when a plugin isn't behaving as expected.
---

# /diagnose

Health check for your BrightWayAI plugin ecosystem. Reports what's set up correctly, what's missing, and what to do about it.

This command is part of `core-ops` because the toolkit plugin is the natural place for cross-cutting diagnostics. You can invoke it whenever something feels broken or whenever you've added/changed plugins and want to confirm everything's wired up.

---

## Step 1 — Check shared config files

These are the canonical files every plugin reads. Check each:

### 1A — Shared identity (`~/Documents/Claude/identity.md`)
- **File exists?**
- **Has all required sections** (Person, Company, Primary tools, Communication defaults)?
- **No placeholder values** (e.g., no `[your name]` left in)?

If missing or incomplete: ✗ → "Run `/setup-identity` (cortex) to capture name/company/role/tools once. Every plugin reads it."

### 1B — Shared voice (`~/Documents/Claude/voice.md`)
- **File exists?**
- **Has voice descriptors, banned phrases, sign-off, hook patterns**?

If missing: ✗ → "Run `/setup-voice` (cortex) to capture writing voice once. Every drafting plugin reads it."

### 1C — Cortex memory (`~/Documents/Claude/memory/DASHBOARD.md`)
- **File exists?**
- **Has at least one node**?
- **`user.md` exists at `~/Documents/Claude/memory/user.md`**?

If missing: ✗ → "Cortex hasn't been initialized. Either install claude-cortex or, if installed, restart Cowork to trigger initialization."

---

## Step 2 — Check plugin setup state

For each marketplace plugin, file paths depend on Cowork's plugin install location, which varies. Instead of probing the filesystem, ask the user.

Run this dialog:

> "I'm going to ask you which plugins you have installed. For each one, I'll verify it's been set up correctly."

Then for each plugin in the marketplace catalog:

| Plugin | Setup command | What to verify |
|---|---|---|
| claude-cortex | (auto) | `/recall` returns something useful |
| core-ops | `/setup-core` | `/pipeline-analyst` invocation works (or `/review-deliverable` runs) |
| lead-engine | `/lead-setup` | `/lead-pipeline` shows an empty or populated pipeline (not an error) |
| bizdev-outreach | `/setup` | `/bizdev-outreach` recognizes a contact name |
| weekly-outreach | `/setup-outreach` | `/weekly-outreach` doesn't error on Step 0 |
| news-curator | `/setup-news` | `/ai-roundup` doesn't error on Step 1 |
| plan-tomorrow | `/setup-plan` | `/plan-tomorrow` doesn't error on Step 0 |
| project-setup | `/setup-projects` | `/project-setup` references a real offering from your catalog |
| weekly-alignment | `/setup` (in skills) | `/scan` doesn't fail on missing channels |

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
| `memory-librarian` | claude-cortex | "Try `/search` with a broad query — does it route to memory-librarian?" |
| `transcript-reviewer` | claude-cortex | "Available; runs on demand or scheduled." |
| `contact-researcher` | lead-engine | "Available to bizdev-outreach, weekly-outreach, lead-engine commands." |
| `pipeline-analyst` | core-ops | "Available to weekly-outreach, plan-tomorrow." |
| `news-curator` (agent) | news-curator | "Used by `/ai-roundup`." |
| `post-assembler` | news-curator | "Used by `/ai-roundup`." |

For each: report whether the parent plugin is installed (per Step 2). If yes → ✓. If no → ✗ "Install [plugin] to make this subagent available."

If the user reports a subagent isn't being invoked when expected, suggest:
1. Restart Cowork (subagent enum sometimes requires reload)
2. Verify the plugin is installed and the agent file is in `[plugin]/agents/`
3. Check the Task tool's `subagent_type` enum — the agent name should appear there

---

## Step 4 — Check connector availability

Connectors (CRM, email, calendar, etc.) are installed via Cowork's connections panel separately from plugins. Per identity.md's "Primary tools" section, ask the user to confirm each is connected:

For each tool listed in `~/Documents/Claude/identity.md` Primary tools section:
- "[Tool name] connected in Cowork? (Y/N)"
- If N → ✗ "Plugins that need [tool] will fall back to inline mode or fail. Connect via Cowork → Connections → [tool]."
- If Y → ✓

Common connectors:
- HubSpot / Salesforce / Pipedrive (CRM)
- Gmail / Outlook (email)
- Google Calendar / Outlook Calendar
- Slack
- Google Drive
- Apollo (for lead-engine)
- Granola (for transcript-reviewer)

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
- **Be specific.** "Identity is missing" is unhelpful. "Identity file at `~/Documents/Claude/identity.md` is missing — run `/setup-identity` (in cortex)" is helpful.
- **Don't be alarmist.** A missing optional file (e.g., voice.md) is fine if the user doesn't draft. Note it as informational, not red.

---

## When this is most useful

- **Onboarding** — first-time setup of the marketplace
- **Something feels broken** — agent isn't invoking, connector seems off
- **After a major change** — you installed/removed plugins, restarted Cowork, switched machines
- **Before a busy day** — quick "is everything wired up?" check before the workflows that depend on it

Run `/diagnose` once a month or whenever you suspect something. Takes ~3 minutes.
