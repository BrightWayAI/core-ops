# core-ops

A generic business-ops toolkit for Claude (Cowork + Claude Code).

Two capabilities, both portable across users and CRMs once configured:

- **Pipeline Analyst** (subagent) — scores and ranks your CRM pipeline by recency × signal strength × stage, returning a prioritized weekly action list.
- **Deliverable Reviewer** (slash command) — runs a structured QA pass on a client deliverable (deck, doc, spreadsheet, one-pager) against your brand guide and the original brief.

## Install

Recommended: install via the [BrightWayAI marketplace](https://github.com/BrightWayAI/claude-plugins).

Or directly:

```
/plugin marketplace add BrightWayAI/core-ops
/plugin install core-ops@core
```

## First-time setup

Run `/setup-core`. The setup skill walks you through a short interview and captures:

- **CRM context** — which CRM you use, which pipeline stages matter, what "good" looks like for prioritization.
- **Brand context** — your brand colors, typography, tone-of-voice rules, and (optionally) a path to your brand guide.
- **Owner / company info** — basic identity so the agents and slash commands can address you correctly.

Answers are saved to `references/user-context.md` (gitignored — never committed to your fork). Agents and skills read from this file before doing real work.

You can re-run `/setup-core` anytime to update.

## What's inside

```
.claude-plugin/plugin.json     Plugin manifest
agents/
  pipeline-analyst.md          Subagent: weekly CRM pipeline ranking
commands/
  review-deliverable.md        Slash command for deliverable QA
  setup-core.md                Interview and config writer
skills/
  setup/SKILL.md               Auto-fires on setup phrases
  review-deliverable/SKILL.md  Auto-fires on review phrases
references/
  user-context.template.md     Structure (committed)
  user-context.md              Your config (gitignored, created by setup)
```

## Dependencies

- A CRM connector available to Claude (HubSpot, Pipedrive, Salesforce — anything Cowork or Claude Code can call). Pipeline Analyst infers structure from properties.
- For Deliverable Reviewer: file access to whatever you're reviewing (Drive connector or local path).

## License

MIT. See `LICENSE`.
