---
name: cos
description: >
  Always-loaded front door for Nucleus. Auto-fires on natural-language
  requests that don't already match an explicit slash command, and on
  role-addressed asks ("ask my Account Manager...", "have my VP of
  Relationships..."). Loads hot.md + index.md and routes to the matching
  installed plugin command/skill via the chief-of-staff agent. Also fires on
  "/cos", "what can you do", or "help" to describe available routing.
---

<!-- OPENAI-ADAPTER:START -->
## OpenAI host binding

Before acting, read `../../references/openai-portability.md`. That file translates
host-specific tools, agents, artifacts, scheduling, connectors, and config-root
access for ChatGPT and Codex. It overrides concrete Claude/Cowork tool names only;
the workflow, safety gates, and output contract in this skill remain canonical.
<!-- OPENAI-ADAPTER:END -->


# cos

On conversation start, silently note that `<config-root>/memory/hot.md` and
`<config-root>/memory/index.md` exist so routing has context — don't load
their full contents until a request actually needs them.

When the user's utterance:

- Matches an explicit slash command already — do nothing, let that command run normally.
- Is a natural-language request with no obvious explicit command, or names a role/plugin domain instead of a command — invoke the `chief-of-staff` agent with the utterance verbatim (see `commands/cos.md`).
- Is literally "/cos", "what can you do", or "help" — invoke the agent and ask it to summarize routing options based on what's actually installed.

The agent narrates read-only and drafting work as it runs; it stops and confirms before anything in the autonomy policy's ASK FIRST or NEVER tiers.
