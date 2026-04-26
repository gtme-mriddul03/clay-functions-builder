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

| Field | Type | Required | Default | Default behavior |
|---|---|---|---|---|
| | | | | |

**Exclusion list:** Is there a category that should always disqualify, regardless of positive criteria?

**Pass-throughs:** What does this Function fetch internally? For each fetch, could a caller already have it? (If yes → optional input that skips the fetch.)

---

## Q4 — Who consumes the output and what do they need?

Name the downstream table, formula, or Function. List only the fields they actually read.

**Consumer:**

**Fields needed:**

---

## Q5 — What comes out?

| Field | Type | Success value | Failure value | Clay column name |
|---|---|---|---|---|
| | | | | |

**Reasoning string needed?** Yes / No — because:

**Confidence score needed?** Yes / No — because:

---

## Q6 — Agent architecture (model fills this — do not leave blank, do not ask the user)

Infer: fetch agents → Claygent (internet); classification agents → LLM (no internet). Independent fetches → parallel. Every fetch has a pass-through input from Q3.

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

## Go / No-Go

- [ ] Q1 has no "and"
- [ ] Q2 has ≥2 named consumers
- [ ] Q3 exclusion list and pass-throughs answered
- [ ] Q4 consumer named before outputs designed
- [ ] Q5 has failure values for every field and Clay column names filled
- [ ] Q6 answered if agents are involved
- [ ] Q7 answered
- [ ] Q8 answered
- [ ] Q9 follows naming-conventions.md

All checked → take to `sop/01-design.md`.
