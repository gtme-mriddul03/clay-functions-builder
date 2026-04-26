# Function Spec — `<function_name>`

<!-- This is the interface contract. Fill it during design. Update it on every change. -->
<!-- One file per Function. Versions are tracked in the Change Log section below. -->

---

## Identity

| Field | Value |
|---|---|
| Name | `<function_name>` |
| Current version | v1 |
| Clay UI name | `<function_name>_v1` |
| One-line description | <!-- verb-first, input→output, no "and" --> |
| Status | `active` / `deprecated — use <replacement>` |

---

## Inputs

| Field | Type | Required | Default | Default behavior |
|---|---|---|---|---|
| <!-- e.g., `url` --> | <!-- e.g., string --> | <!-- e.g., yes --> | <!-- e.g., — --> | <!-- e.g., — --> |
| <!-- e.g., `depth` --> | <!-- e.g., enum: basic, full --> | <!-- e.g., no --> | <!-- e.g., basic --> | <!-- e.g., HTTP check only; skip parked detection --> |

---

## Outputs

| Field | Type | Success value | Failure value |
|---|---|---|---|
| <!-- e.g., `is_valid` --> | <!-- e.g., boolean --> | <!-- e.g., true --> | <!-- e.g., false --> |
| <!-- e.g., `canonical_url` --> | <!-- e.g., string --> | <!-- e.g., "https://acme.com" --> | <!-- e.g., null --> |

---

## Behavior notes

<!-- Edge cases not obvious from the tables above. -->
<!-- e.g., "If `depth` is `basic`, `canonical_url` is always null regardless of resolution." -->
<!-- Delete this section if there are no non-obvious edge cases. -->

---

## Composition

<!-- Does this Function call sub-Functions? List them here with versions. -->
<!-- If this Function is a standalone: delete this section. -->

| Sub-Function | Version used | Why |
|---|---|---|
| | | |

---

## Change log

| Version | Date | Type | What changed |
|---|---|---|---|
| v1 | <!-- YYYY-MM-DD --> | initial | Initial publish |

<!-- Type values: "initial", "additive", "breaking" -->
<!-- Add a row here for every change. Breaking changes also need an ADR in decisions/. -->

---

## Known consumers

<!-- List every Clay table or workflow referencing this Function. -->
<!-- Update this when you add or remove a consumer. -->
<!-- This list matters most when you cut a new version. -->

- 

---

## Related

<!-- Link to usage doc and any relevant ADRs -->

- Usage doc: `docs/<function-name>-usage.md`
- ADRs: <!-- link to decisions/ entries, or "none" -->
