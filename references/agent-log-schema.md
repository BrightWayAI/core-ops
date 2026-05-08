# Agent Run Log Schema

The agent-run log lives at `~/.brightway-state/agent-log.jsonl`.

JSON Lines format — one JSON object per line. Append-only. Plain-text, grep-friendly, backup-friendly.

## Schema

```json
{
  "timestamp": "2026-05-08T14:32:00Z",
  "agent": "contact-researcher",
  "parent_skill": "bizdev-outreach",
  "confidence": "Medium",
  "user_action": "iterated",
  "latency_seconds": 45,
  "notes": "Dossier missed recent LinkedIn activity"
}
```

### Field definitions

| Field | Type | Required | Notes |
|---|---|---|---|
| `timestamp` | ISO 8601 UTC | yes | When the agent run completed (or when the log entry was written, if asynchronous) |
| `agent` | string | yes | The subagent name (matches the agent's frontmatter `name` field) |
| `parent_skill` | string | yes | The skill or command that invoked the agent (e.g., `bizdev-outreach`, `/lead-brief`, `/weekly-outreach`) |
| `confidence` | enum | yes | The Confidence rating returned by the agent: `High`, `Medium`, `Low` |
| `user_action` | enum | yes | What the user did with the agent's output: `accepted` (used as-is), `iterated` (modified before using), `overrode` (changed substantially), `abandoned` (didn't use the output, took over manually) |
| `latency_seconds` | int | optional | Roughly how long the agent took (from invocation to return) |
| `notes` | string | optional | Free-form qualitative — anything worth remembering. ≤200 chars recommended. **Never** include sensitive content (names, message bodies, CRM data) |

### Reserved values

- `parent_skill` for direct/manual invocations: use `"manual"` (e.g., user invoked the agent directly via the Task tool, not through a skill)
- `user_action` for headless / scheduled runs where there's no immediate user: use `"scheduled"` and a separate review entry can capture later evaluation
- `notes` to flag known issues: prefix with `BUG:`, `LIMITATION:`, or `UPSTREAM:` for searchability

## Privacy

This log is **meta-only**. It records *what happened* (which agent, which skill, what confidence, what action), not *what the data was*.

**Do not log:**
- Contact names, emails, or any CRM/customer data
- Message content (drafted, sent, or received)
- Memory entries or cortex node names that contain personal/client info
- File paths if they contain sensitive directory names

**Safe to log:**
- Agent and skill names (those are public marketplace identifiers)
- Confidence and action enums
- Generalized notes ("dossier missed recent activity," "stage probabilities seemed off") — without naming the contact or company

If in doubt, redact.

## Append-only

The log is append-only by design:

- `/log-agent-run` only appends.
- `/agent-metrics` only reads.
- The user can manually edit the file (it's plain text), but the plugins won't.
- For cleanup, the user can manually rotate (e.g., move old entries to an archive file). The plugins don't do automatic rotation.

## Backup

Plain-text JSONL. Back up however you back up `~/.brightway-state/`. Combined with `~/Documents/Claude/` (memory, identity, voice, time-log), that's your full BrightWayAI plugin state.

## Volume expectations

A user with the full marketplace stack who logs every notable run should produce 5–20 entries per workday. Over a year, that's 1,500–7,500 lines — totally fine for grep + JSONL parsing. If it grows beyond 50K lines, manual rotation makes sense.

## Sample entries

```jsonl
{"timestamp":"2026-05-08T09:15:00Z","agent":"pipeline-analyst","parent_skill":"weekly-outreach","confidence":"Medium","user_action":"accepted","latency_seconds":32,"notes":""}
{"timestamp":"2026-05-08T11:42:00Z","agent":"contact-researcher","parent_skill":"bizdev-outreach","confidence":"High","user_action":"iterated","latency_seconds":58,"notes":"Talking points were good but tone needed sharpening"}
{"timestamp":"2026-05-08T14:03:00Z","agent":"memory-librarian","parent_skill":"/search","confidence":"Low","user_action":"abandoned","latency_seconds":18,"notes":"BUG: missed an entry that exists in cortex memory; investigate"}
{"timestamp":"2026-05-08T16:20:00Z","agent":"news-curator","parent_skill":"/ai-roundup","confidence":"Medium","user_action":"accepted","latency_seconds":124,"notes":"Slow week — only 6 strong candidates"}
```

Sparse, qualitative, surgical. That's the bar.
