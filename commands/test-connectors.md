---
description: Run bounded, read-only calls against real authorized connectors and return a sanitized evidence report for Nucleus release certification. Use for connector smoke tests, pre-release checks, or proving that calendar, mail, CRM, Slack, Drive, enrichment, and transcript integrations actually work.
---

# /test-connectors

Test real connector access instead of asking the user whether a connection appears to
be configured. This workflow is read-only and produces metadata-only evidence that can
be checked by the Nucleus release tooling.

## Step 0 — Establish the test boundary

Ask for these inputs in one compact prompt:

1. Nucleus release ID, such as `2026.09.0-rc.1`.
2. Host: `claude`, `chatgpt`, `codex`, or `direct-mcp`.
3. Profile:
   - `operator` — calendar, mail, CRM;
   - `collaboration` — Slack, Drive;
   - `research` — contact enrichment, transcripts;
   - `full` — all seven.
4. The designated sandbox mailbox, calendar window, CRM record/query, Slack channel,
   Drive folder, synthetic enrichment target, or transcript source to use.

Show the selected probes and obtain confirmation before reading any external system.
If the user cannot identify a safe test target, mark that connector `skip`; do not
quietly query their general production data.

V1 never performs write probes. Do not create events, send mail, mutate CRM, post to
Slack, upload files, or alter connector configuration.

## Step 1 — Discover actual tools

For every connector in the selected profile, inspect the tools available in the active
host and select a read operation capable of the bounded probe below. Record the exact
tool identifier. Do not infer availability from setup files or the user's memory.

| Connector ID | Required live probe | Used by |
|---|---|---|
| `calendar` | List at most one event in the approved narrow window | daily-brief, time-tracking, delivery, relationships |
| `mail` | Search the approved test mailbox or marker with limit 1 | daily-brief, delivery, relationships |
| `crm` | Read the designated sandbox record or run an approved bounded empty query | core-ops, delivery, relationships |
| `slack` | Search the designated test channel with a bounded query | weekly-alignment, daily-brief |
| `drive` | List at most one item from the designated test folder | delivery, daily-brief |
| `contact-enrichment` | Look up the designated synthetic company/contact | relationships |
| `transcripts` | List at most one record from the designated test source | cortex |

If no compatible read tool exists, record `skip` with error code
`connector_unavailable`. Authentication or authorization failures are `fail`, not
`skip`.

## Step 2 — Execute bounded reads

Call each discovered connector exactly once unless the first response is a documented
pagination envelope that requires one non-content follow-up to determine the record
count. Measure latency when the host exposes timing; otherwise use `null`.

A connector passes only when a real tool call succeeds and its response shape is usable
by the dependent plugins. Empty search results may pass when the designated probe was
expected to be empty. A pasted payload, mocked response, cached prose description, or
user assertion can never produce `pass`.

Inspect the response only long enough to retain:

- record count;
- content block types;
- top-level response keys.

Do not reproduce subjects, names, message text, event titles, file names, contact
details, transcript text, record values, or connector payloads in the report.

## Step 3 — Return the sanitized report

Return one JSON object with this exact shape:

```json
{
  "schemaVersion": 1,
  "release": "2026.09.0-rc.1",
  "host": "claude",
  "profile": "operator",
  "runAt": "2026-09-15T12:00:00Z",
  "results": [
    {
      "id": "calendar",
      "status": "pass",
      "tool": "exact.tool.identifier",
      "latencyMs": 240,
      "observation": {
        "recordCount": 1,
        "contentTypes": ["structured"],
        "topLevelKeys": ["events", "nextPageToken"]
      },
      "errorCode": null
    }
  ]
}
```

Include one result for every connector in the canonical seven-connector plan. Use
`not-run` for connectors outside the selected profile. Use stable, non-sensitive error
codes such as `connector_unavailable`, `authentication_failed`, `authorization_failed`,
`timeout`, `tool_error`, or `schema_incompatible`.

The report is conversation output by default. Save it only when the user explicitly
chooses a path after reviewing it. For release evidence, validate it in the Nucleus
checkout with:

```bash
python3 scripts/validate_connector_report.py /path/to/report.json \
  --release <release-id> \
  --profile <profile>
```

## Behavior rules

- Real call or no pass.
- Read-only means read-only; never broaden this workflow into mutation testing.
- Use sandbox targets and the smallest query possible.
- Never put raw connector content or credentials in a report.
- Run separately per host. A Claude pass does not prove ChatGPT authorization, and a
  ChatGPT pass does not prove Claude authorization.
- Report partial failures honestly and identify the dependent plugins affected.
