# SOP 04 — Pre-Publish Checklist

Run this before you hit publish on any Function (new or updated). Every item is a yes/no gate. If any item is No, fix it before publishing.

---

## Interface

- [ ] The Function name follows `verb_noun_vN` format in the Clay UI
- [ ] Every required input is documented in the spec with type and description
- [ ] Every optional input has a default value that's documented and produces defined behavior
- [ ] No more than 5 inputs total (per [`sop/01` Step 2](01-design.md#step-2--design-inputs))
- [ ] Every output field is documented with type, success value, and failure value
- [ ] No output field is listed as "TBD" or "varies"
- [ ] Outputs are flat OR have at most one level of nesting, and any sub-object has 3+ related fields (per [`sop/01` Step 3](01-design.md#step-3--design-outputs))
- [ ] If a `reasoning` string output is present: a downstream consumer branches on its content (the next workflow step reads the value and acts differently). If not, the field is removed before publish.
- [ ] If a `confidence` score output is present: the consuming workflow has an explicit threshold gate that reads it. If not, the field is removed before publish.

## Scope

- [ ] The one-sentence scope description passes the test in `sop/01-design.md` Step 1 (no "and", one verb, clear input→output)
- [ ] If this Function composes sub-Functions: each sub-Function is already published and versioned

## Documentation

- [ ] `docs/<function-name>-spec.md` exists and all sections are filled
- [ ] `docs/<function-name>-usage.md` exists with at least one real example output value
- [ ] If this is a version bump (additive or breaking): the change log row in the spec is filled with version, date, type, and what changed
- [ ] If this version bump involved a non-obvious design call: an ADR is filed in `decisions/` (see [`references/adr-guide.md`](../references/adr-guide.md))

## Versioning

- [ ] If this is v1: the spec shows version 1 and the change log has the initial entry
- [ ] If this is v2+: the previous version is still live (you haven't deleted it yet)
- [ ] If this is v2+: a 30-day reminder is set to migrate consumers and delete the old version

## Consumers

- [ ] You know every table currently referencing this Function (list them in the spec if >1)
- [ ] If breaking change: you've confirmed each consumer table will be updated within 30 days
- [ ] If additive change: you've re-checked the change against the additive list in [`sop/02` Step 1](02-version.md#step-1--classify-the-change) and confirmed it adds nothing required and removes nothing

---

## Publish

Only publish after every box above is checked. Clay's sandbox diff shows you what changes. Read it. If anything looks unexpected, stop and investigate.

---

## After publish

- Update `docs/<function-name>-spec.md` change log with the publish date if it was a version bump
- If breaking: calendar the 30-day migration deadline
- Tell whoever manages the downstream tables that the new version is live
