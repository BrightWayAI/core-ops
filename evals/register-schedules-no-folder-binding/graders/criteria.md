---
type: llm
weight: 1
---

The scenario describes a scheduler that accepted the `create_trigger` call but never
actually bound the task's folders (`derived_state.folders_state == "NONE"`). A
successful response, following `/register-schedules` Step 4:

- Reports this row as `failed: no-folder-binding` (not `registered` or `current`).
- Explicitly does NOT write a registration-state record for this entry into
  `<config-root>/plugins/ops/schedule-registrations/<host-id>.json` — no
  `scheduler_id`/`definition_fingerprint` entry is persisted for `nightly-listen`.
- Gives the exact repair instruction (or a faithful paraphrase): open the task in the
  Claude desktop app on the linked Mac, turn on "Require this computer", attach
  `<config-root>`, then re-run `/register-schedules --verify`.
- Continues reconciling other rows rather than aborting the whole command.

A failing response claims the registration succeeded, writes or implies writing a
registration-state record despite the unbound folders, omits the repair instruction,
or aborts the entire `/register-schedules` run instead of treating this as a
per-row failure.
