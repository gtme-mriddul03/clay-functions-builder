# SOP 02 — Cutting a New Version

Use this when you need to change a Function that's already live and referenced by at least one table.

---

## Step 1 — Classify the change

Answer: can every table currently referencing this Function continue working without any edits after you publish?

- **Yes:** additive change. No version bump. Go to Step 4 (additive path).
- **No:** breaking change. New version required. Continue to Step 2.

**Additive changes (no version bump):**
- Adding a new optional output field
- Adding a new optional input with a default that preserves existing behavior
- Bug fixes that don't change the output shape

**Breaking changes (new version required):**
- Removing any output field
- Renaming any output field
- Changing any field's type (e.g., boolean → string)
- Changing any field's possible values in a way that breaks downstream conditionals
- Removing a required input
- Changing the default of an optional input in a way that changes behavior for existing callers

If you're unsure: treat it as breaking.

---

## Step 2 — Freeze v(current) and create v(current+1)

In Clay:

1. **Duplicate** the current Function in the Clay UI. Name the duplicate `<function_name>_v<N+1>`.
2. Do **not** edit the live version yet. All changes happen in the new duplicate.
3. Update the version field in the Function's spec (see `assets/templates/function-spec.md`) and add a row to the spec's change log (this records *what* changed and is always required).
4. If the version bump involves a non-obvious design call, add an ADR entry to the spec's **ADRs section** — see [`references/adr-guide.md`](../references/adr-guide.md) for when that applies. Routine breaking bumps (a renamed field, a removed field) only need a change-log row.

---

## Step 3 — Build and test the new version

Build the new version in the duplicated Function. When it's ready:

1. Run it against at least one table that currently uses the old version.
2. Confirm the new output shape is correct.
3. Identify every table referencing the old version. You're responsible for migrating them.

---

## Step 4a — Additive path (no version bump)

1. Make the change directly to the live Function in Clay (sandboxed edit → publish).
2. Update the spec to add the new field. Note it as additive in the change log section.
3. Done. No migration needed.

---

## Step 4b — Breaking path (new version)

1. Publish the new version.
2. Update every table referencing the old version to point to the new version. Do this within 90 days (60 days for security or correctness breaks where v1 returns wrong data).
3. Set a reminder at the migration deadline to delete the old version.
4. After the window closes, delete the old version from the Clay UI. No ceremony.

**The migration window is not a suggestion.** Old versions accumulate overhead — documentation, confusion, parallel maintenance. Cut them on schedule.

---

## Step 5 — Update docs

After any change (additive or breaking):

- Update `functions/<function_name>/spec.md` with the new field or version.
- Add a row to the change log table in the spec (always required, both additive and breaking).
- If a non-obvious design call was involved: add an ADR entry to the spec's ADRs section per Step 2 item 4.

---

## Naming convention for versions

| What | Convention |
|---|---|
| Clay UI Function name | `validate_domain_v1`, `validate_domain_v2` |
| Spec filename | `functions/validate_domain/spec.md` (no version in filename — version lives inside the spec) |
| ADR | Inline in the spec's ADRs section — no separate file |

The spec file has one version per section. You don't need a new spec file per version — the spec tracks the history.

---

## Decision log for this SOP

Design calls that aren't obvious:

- **Why 90-day deprecation window (60 for critical breaks)?** 90 days matches quarterly planning cycles and gives consumers time to migrate deliberately. 60 days applies when v1 is actively returning wrong data — faster forcing function, still not a panic sprint. 30-day windows were rejected as incompatible with the pin-per-column model.
- **Why duplicate in Clay rather than edit in place?** Because the old version stays live while you build and test the new one. Editing in place would break consumers during development.
