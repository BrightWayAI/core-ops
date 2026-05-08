---
description: Configure core-ops for your CRM, brand, and company context via a short interview. Writes results to references/user-context.md so the pipeline-analyst agent and review-deliverable command can do real work. Re-run anytime to update.
---

# /setup-core

Short interview that captures the context the core-ops agents and commands need to actually be useful for *you* (not just BrightWay).

---

## Step 1 — Check for existing config

Read `references/user-context.md` if it exists.

- If it exists and is populated → ask: "You've already configured core-ops. Want to update specific sections, or start over?"
  - "Update [section]" → jump to that section, ask only those questions, write back.
  - "Start over" → continue full interview.
- If it doesn't exist → start fresh. Read `references/user-context.template.md` for the structure to populate.

---

## Step 2 — The interview

Ask one section at a time. After each section, summarize what you heard and ask the user to confirm or correct before moving on. Don't bombard with all questions at once.

### Section 1 — Identity (basics)

- Your name
- Your company name
- Your role / title
- A one-sentence description of what your company does

### Section 2 — CRM context

- Which CRM do you use? (HubSpot / Pipedrive / Salesforce / Attio / Affinity / other / "I don't have one")
- If "I don't have one" → note that pipeline-analyst won't be useful until they connect one. Skip the rest of this section.
- What are your pipeline stages, in order from earliest to latest? (e.g., "Lead, Qualified, Demo, Proposal, Negotiation, Closed Won")
- Which stages do you weight most heavily for prioritization? (Usually the late-stage ones — Proposal, Negotiation, etc.)
- What does "good" pipeline activity look like for you? (e.g., "two-way email exchange in last 14 days," "demo booked," "proposal sent within 7 days of qualification")
- Any stages or contact types you want excluded from analysis? (e.g., "ignore Customer-stage accounts unless they're up for renewal")

### Section 3 — Brand context (for /review-deliverable)

- What are your brand colors? (Hex codes if you have them, names if not)
- What typefaces do you use for headings and body?
- Three words that describe your brand voice. (e.g., "warm, direct, low-jargon")
- Any words or phrases you ban in your work? (e.g., "synergy," "leverage," "delight")
- Where is your canonical brand guide? (Drive URL, Notion page, local path, or "I don't have one yet")
- Anything specific about how you format different deliverable types? (e.g., "decks open with an agenda on slide 2," "one-pagers always have a CTA in the footer")

### Section 4 — Optional preferences

- How do you want priorities ranked when items tie? (e.g., "freshness wins," "high-value deal wins")
- Do you have a weekly cadence you're trying to fit into? (e.g., "I do pipeline review Mondays at 8am — surface 5–10 items, not more")

---

## Step 3 — Write the config

Populate `references/user-context.md` with the answers, using the template structure. Format clearly — agents and commands will read this file every invocation, so structure it for fast reading:

```markdown
# core-ops user context

_Last updated: [date]_

## Identity
- **Name:** ...
- **Company:** ...
- **Role:** ...
- **What we do:** ...

## CRM
- **Tool:** ...
- **Stages (ordered):** ...
- **Weighted stages:** ...
- **Definition of "good":** ...
- **Excluded:** ...

## Brand
- **Colors:** ...
- **Typefaces:** ...
- **Voice:** ...
- **Banned phrases:** ...
- **Brand guide:** ...
- **Format conventions:** ...

## Preferences
- **Tiebreaker:** ...
- **Cadence:** ...
```

---

## Step 4 — Confirm and offer next step

After writing, summarize what was saved (one short paragraph) and offer a concrete next step:

- If CRM is configured → "Try `/pipeline-analyst` (via `weekly-outreach` or directly via the Task tool) to see your prioritized list."
- If brand is configured → "Try `/review-deliverable [path]` on a recent draft."
- If both → both above, plus "Or just keep working — both will use this context automatically."

---

## Behavior rules

- **One section at a time.** Don't paste all 25 questions at once.
- **Skip what doesn't apply.** If the user doesn't have a brand guide, "no brand guide yet" is a valid answer — capture it as such.
- **Don't pad answers.** If the user says "skip Section 4," skip it. Note in user-context that it was skipped so the agents know to use defaults.
- **Idempotent.** Running `/setup-core` again should let the user update sections without re-doing everything.
- **Privacy-respecting.** Everything written goes to `references/user-context.md`, which is gitignored — never committed to a fork.
