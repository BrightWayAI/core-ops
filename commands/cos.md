---
description: Talk to your chief of staff. Natural-language front door for Nucleus — routes any request or role-addressed ask ("ask my Account Manager to draft the status update") to the right installed plugin command, narrating read-only work and confirming before anything that writes, sends, or spends. Replaces nucleus-router.
---

# /cos

Invoke the `chief-of-staff` agent (`agents/chief-of-staff.md`) with the user's request verbatim as input.

The agent reads `<config-root>/memory/hot.md` and `<config-root>/memory/index.md` first, then routes to the matching installed command/skill/agent. It follows the autonomy policy in `<config-root>/memory/CLAUDE.md` for anything beyond read-only or drafting work.

Plain natural language in a Nucleus session should also reach this agent without the user needing to type `/cos` explicitly, wherever the host supports always-loaded skill routing — see `skills/cos/SKILL.md`.

If no plugin covers the request, the agent says so and names the missing capability rather than improvising a workaround.
