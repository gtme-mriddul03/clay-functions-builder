# Function Canvas

Copy this file to `working/canvas-<function-name>.md`. Fill it out *before* building. It's a thinking tool, not a doc.

---

## Q1 — What does this Function do? (one sentence, verb-first)

<!-- e.g., "Takes a raw URL and returns whether it resolves to a live, non-parked domain." -->
<!-- No "and." If you need "and," split. -->

**Answer:**

---

## Q2 — Who uses this? (tables and workflows, not people)

<!-- List every Clay table or workflow that will reference this Function.
     If you can only name one and no second is scheduled, stop. Go build it inline. -->

**Current consumers:**

**Upcoming consumers:**

---

## Q3 — What goes in?

<!-- For each input: name, type, required?, default value and what it does -->
<!-- Max 5 inputs. If you need more, scope is wrong. -->

| Field | Type | Required | Default | Default behavior |
|---|---|---|---|---|
| | | | | |

---

## Q4 — What comes out?

<!-- For each output: name, type, value on success, value on failure -->
<!-- Flat unless natural sub-object. No nested nesting. -->
<!-- Include reasoning string? Only if a downstream consumer branches on it. -->
<!-- Include confidence score? Only if a downstream threshold gate reads it. -->

| Field | Type | Success value | Failure value |
|---|---|---|---|
| | | | |

**Reasoning string needed?** Yes / No — because:

**Confidence score needed?** Yes / No — because:

---

## Q5 — Is anything in here independently useful to a different Function?

<!-- If yes, name the other Function and the shared primitive. -->
<!-- If no, all logic stays inside this Function. -->

**Answer:**

---

## Q6 — What's the one failure mode we need to handle explicitly?

<!-- e.g., "Input URL is empty string" / "Domain redirects to a different registrable domain" -->
<!-- Define what the output is in that case. -->

**Failure case:**

**Output in that case:**

---

## Go / No-Go

After filling Q1–Q6, check:

- [ ] Q1 has no "and"
- [ ] Q2 has at least 2 consumers, or one incoming this week (confirmed, not speculative — name the table or workflow)
- [ ] Q3 has ≤5 inputs
- [ ] Q4 has failure values for every field
- [ ] Q5 is answered (not blank)
- [ ] Q6 is answered (not blank)

All checked? Take this canvas to `sop/01-design.md` Step 1 and lock the interface.
