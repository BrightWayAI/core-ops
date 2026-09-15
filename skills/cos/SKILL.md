---
name: cos
description: >
  Nucleus front door for natural-language work. Fires on explicit /cos,
  Nucleus-domain requests that don't already match a slash command, and on
  role-addressed asks ("ask my Account Manager...", "have my VP of
  Relationships..."). Uses the chief-of-staff agent to produce a read-only,
  structured route plan; the parent validates and executes the selected
  installed workflow. Also fires on Nucleus-specific capability questions.
---

<!-- OPENAI-ADAPTER:START -->
## OpenAI host binding

Before acting, read `../../references/openai-portability.md`. That file translates
host-specific tools, agents, artifacts, scheduling, connectors, and config-root
access for ChatGPT and Codex. It overrides concrete Claude/Cowork tool names only;
the workflow, safety gates, and output contract in this skill remain canonical.
<!-- OPENAI-ADAPTER:END -->


# cos

On conversation start, discover the installed plugin manifest set. Do not read
Cortex memory merely to decide whether a request belongs to Nucleus.

When the user's utterance:

- Matches an explicit slash command already — do nothing, let that command run normally.
- Is a Nucleus-domain request with no obvious explicit command, or names a Nucleus role/plugin domain instead of a command — invoke the `chief-of-staff` agent with the utterance verbatim (see `commands/cos.md`).
- Is `/cos`, "what can Nucleus do?", or "help me use Nucleus" — invoke the agent and ask it to summarize routing options based on what's actually installed.
- Is generic conversation or a generic "help" request with no Nucleus signal — do not capture it. Let the host or another installed plugin handle it.

The agent never executes work. It returns a structured route plan. The parent
validates that the target is installed, loads only the context named by the plan,
and owns every execution and autonomy gate.
