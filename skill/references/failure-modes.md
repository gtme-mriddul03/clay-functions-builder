# Failure Modes — What This Framework Guards Against

A catalog of the ways Clay Functions go wrong. The SOPs in this framework exist to prevent these.

---

## FM-01 — Silent breaking change

**What happens:** You rename an output field in a live Function. Every table referencing it silently fails to enrich or returns empty cells. You find out days later.

**Why it happens:** Clay doesn't enforce interface contracts. There's no "breaking change" warning.

**Guard:** SOP 02 versioning rules. Any output field rename = new version. Old version stays live for 30 days. Consumers migrate on their own timeline.

---

## FM-02 — Scope creep without a split

**What happens:** A Function grows to check five different things. It becomes hard to maintain, slow to run, and impossible to reuse any single check elsewhere.

**Why it happens:** It's easy to add "one more check" to an existing Function instead of evaluating whether it belongs.

**Guard:** SOP 01 Step 1 one-sentence test. If you can't describe the Function's scope in one sentence without "and," the scope has already crept. Split before adding.

---

## FM-03 — Premature extraction into primitives

**What happens:** You build `normalize_url`, `check_https`, `check_parked` as separate Functions. Each needs its own docs, versioning, and management overhead. None is used by more than one other Function.

**Why it happens:** Primitives feel clean and reusable even when nothing actually reuses them.

**Guard:** SOP 01 Step 4 composition check. Extract a primitive only when the second concrete use is in-hand, not anticipated.

---

## FM-04 — Useless reasoning strings

**What happens:** Every Function output includes a `reasoning` field. Nobody reads it. It adds noise to Clay cells and costs tokens in Claygent.

**Why it happens:** "It might be useful" is enough justification when there's no decision rule.

**Guard:** SOP 01 Step 3 reasoning string gate. Include only if a downstream consumer branches on the text. If nobody reads it to make a decision, it doesn't exist.

---

## FM-05 — Boolean status that can't represent reality

**What happens:** `is_valid: boolean` is the output. Six months in, you need to distinguish "parked" from "unreachable" from "redirect mismatch." You can't — the field is a boolean. Breaking change required.

**Why it happens:** Boolean feels simpler at design time. You don't think through all the states.

**Guard:** SOP 01 Step 3 output design. If you can imagine more than two meaningful states, use an enum from the start. Name the states explicitly.

---

## FM-06 — Version pileup

**What happens:** v1, v2, and v3 all live in parallel. Nobody knows which one to use. The docs reference v1 but the new hire builds on v2. Some tables are on v1, some on v2, and nobody's sure if v3 is ready.

**Why it happens:** Old versions are never deleted. There's no deprecation deadline.

**Guard:** SOP 02 30-day deprecation window. After 30 days, old versions are deleted. Non-negotiable.

---

## FM-07 — Canvas skipped, interface discovered during build

**What happens:** You start building the Function directly in Clay. The output shape changes three times during development. By the time it's live, no spec exists and nobody knows what the output is.

**Why it happens:** The canvas feels like overhead before the first build.

**Guard:** SOP 01 is explicitly before building. The spec is written during design, not after. SOP 04 pre-publish checklist requires the spec to exist before publish.

---

## FM-08 — Single consumer that could have stayed inline

**What happens:** A Function is built for one table. It accumulates documentation and versioning overhead. The table is rebuilt six months later and the Function is never used again.

**Why it happens:** Building a Function feels like the "right" thing even for single-use patterns.

**Guard:** SOP 00 Gate 1. One table, no second consumer scheduled = inline logic. Come back when you hit a second real use.
