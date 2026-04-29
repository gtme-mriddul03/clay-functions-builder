# Function Spec — `<function_name>`

---

## Identity

| Field | Value |
|---|---|
| Name | `<function_name>` |
| Current version | v1 |
| Clay UI name | `<function_name>_v1` |
| One-line description | |
| Status | `active` |

---

## Inputs

Use Clay UI names (Title Case) — these are the exact names as they appear in the Clay Function UI.

| Clay UI Name | Type | Required | Default | Default behavior | Clay description |
|---|---|---|---|---|---|
| | | | | | |

**Clay description:** short phrase pasted directly into Clay's input description field (visible to anyone adding this Function to a table).

---

## Exclusions

Optional inputs that disqualify records regardless of positive criteria. Delete this section if none apply.

| Clay UI Name | Type | Default | What it excludes |
|---|---|---|---|
| | | `null` | |

Removing or redefining an exclusion input triggers a version bump. Adding a new optional exclusion input (default: null) is additive — it doesn't change behavior for callers who don't pass it.

---

## Outputs

| Clay UI Name | Type | Success value | Failure value | Clay type |
|---|---|---|---|---|
| | | | | |

Clay types: Text, Number, Boolean, JSON, Date

---

## Clay column names

Reference when building in Clay. Every row here corresponds to something you name in the Clay UI. See `references/naming-conventions.md`.

| Field | Clay name | Kind | Convention applied |
|---|---|---|---|
| | | input / output / internal | |

Kind values: `input` (Function input field), `output` (output field or column consuming it), `internal` (agent or formula column inside the Function).

---

## Agent architecture

Fill if this Function uses agents or load-bearing formula columns. Delete if standalone.

| Name | Type | Internet | Runs after | Skipped when |
|---|---|---|---|---|
| | Claygent / LLM / Formula | yes / no | | |

Type guide: `Claygent` = web-research agent; `LLM` = classification/extraction with no internet; `Formula` = any formula column whose output is read by another row or consumed as a Function output — conditional gates, consolidation formulas, URL reconstruction. Exclude incidental helper formulas with no downstream reader.

---

## Behavior notes

Edge cases not obvious from the tables above. Delete if none.

---

## Composition

Sub-Functions called (delete if standalone):

| Sub-Function | Version | Why |
|---|---|---|
| | | |

---

## Version increment triggers

A v2 is cut when: (fill at design time)

---

## Change log

| Version | Date | Type | What changed |
|---|---|---|---|
| v1 | YYYY-MM-DD | initial | Initial publish |

Type values: `initial`, `additive`, `breaking`. Breaking changes also noted as ADRs below.

---

## ADRs

Non-obvious design decisions made during this Function's lifetime. One entry per decision.

---

## Known consumers

- 

---

## Related

- Usage doc: `functions/<function_name>/usage.md`
- Backlog: `functions/<function_name>/backlog.md`
