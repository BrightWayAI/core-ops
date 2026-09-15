# Schedule Library Reference

The bundled schedule starter is `schedules.template.md`. Runtime definitions and
state never live in this installed plugin directory.

## User-owned paths

- Definitions: `<config-root>/plugins/core-ops/schedules.md`
- Per-host IDs: `<config-root>/plugins/core-ops/schedule-registrations/<host-id>.json`
- Run receipts: `<config-root>/plugins/core-ops/schedule-runs/<schedule>/<run-id>.json`

Run `/register-schedules` to copy the starter, validate installed dependencies,
reconcile this host's scheduler, preview changes, and request confirmation.

## Nightly listen prompt requirements

The `nightly-listen` registration must include all of these constraints:

1. Exit without writes if the delayed trigger starts after 06:00 local time.
2. Run Cortex `/listen` for yesterday using the resolved `<config-root>`.
3. Read enabled sources from the Cortex source configuration.
4. Write only staged proposals and refreshed derived caches; never durable memory
   nodes during an unattended run.
5. Never block for permission. Record a sanitized permission-missing code and exit
   partial or failed.
6. Emit a metadata-only run receipt using the schema in
   `commands/register-schedules.md`.

Before registration, run `/listen` manually once so the human can grant each desired
connector independently. A schedule definition is not proof that registration or a
successful run occurred.
