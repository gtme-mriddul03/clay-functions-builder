# Function Canvas

Copy to `functions/<verb_noun>/canvas.md`. Fill before building. Thinking tool, not a doc.

---

## Q1 — What does this Function do? (one sentence, verb-first, no "and")

**Answer:**

---

## Q2 — Who uses this?

List every Clay table or workflow that will reference this Function. If you can only name one and no second is scheduled, stop — build it inline instead.

**Current consumers:**

**Upcoming consumers:**

---

## Q3 — What goes in?

Use Clay UI names (Title Case) — the exact names as they'll appear in the Clay Function UI.

| Clay UI Name | Type | Required | Default | Default behavior | Clay description |
|---|---|---|---|---|---|
| | | | | | |

**Clay description:** one short phrase per field (≤ 15 words) — pasted into Clay's input description field, visible to anyone adding this Function to a table.

**Exclusion list:** Is there a category that should always disqualify, regardless of positive criteria?

**Pass-throughs:** What does this Function fetch internally? For each fetch, could a caller already have it? (If yes → optional input that skips the fetch.)

---

## Q4 — Who consumes the output and what do they need?

Name the downstream table, formula, or Function. List only the fields they actually read.

**Consumer:**

**Fields needed:**

---

## Q5 — What comes out?

| Clay UI Name | Type | Success value | Failure value | Clay type |
|---|---|---|---|---|
| | | | | |

Clay types: Text, Number, Boolean, URL, Date

**Reasoning string needed?** Yes / No — because:

**Confidence score needed?** Yes / No — because:

---

## Q6 — Agent architecture (model fills this — do not leave blank, do not ask the user)

Infer: fetch agents → Claygent (internet); classification agents → LLM (no internet); conditional gates, consolidation, or URL reconstruction → Formula. Independent fetches → parallel. Every fetch has a pass-through input from Q3.

**Inferred design:**

---

## Q7 — Is anything here independently useful to a different Function?

**Answer:**

---

## Q8 — Primary failure case

**Failure case:**

**Output in that case:**

---

## Q9 — Clay column names

What will the Clay column calling this Function be named? See `references/naming-conventions.md`.

**Function column:**

**Key output columns:**

---

## Q10 — What's deferred and why?

List any improvements, edge cases, or scope items that came up during design but are explicitly not in v1. Note the reason for each deferral. This seeds `functions/<verb_noun>/backlog.md`.

**Deferred items:**

---

## Go / No-Go

- [ ] Q1 has no "and"
- [ ] Q2 has ≥2 named consumers
- [ ] Q3 uses Clay UI names (Title Case); Clay descriptions filled; exclusion list and pass-throughs answered
- [ ] Q4 consumer named before outputs designed
- [ ] Q5 has failure values for every field and Clay column names filled
- [ ] Q6 answered if agents or formula columns are involved
- [ ] Q7 answered
- [ ] Q8 answered
- [ ] Q9 follows naming-conventions.md
- [ ] Q10 answered (even if empty — "nothing deferred" is a valid answer)

All checked → take to `sop/01-design.md`.
