# Practitioner POV — memo

## Q1 — Build gates

A pattern earns promotion when I've copy-pasted it into a third table and the formulas are starting to drift between copies. Not before. Two uses is coincidence; three is a maintenance bill coming due.

Hard gates, in order:

1. **Used in ≥3 tables or ≥2 workspaces.** One-off logic stays as a formula column.
2. **Stable inputs.** If I'm still arguing with myself about whether the input is `domain` or `website_url`, the pattern isn't ready. Functions calcify their interface the second someone else consumes them.
3. **Deterministic enough to cache.** A Function whose output changes every run because it hits a flaky scraper is a credit bonfire disguised as reuse.

**What should NOT be a Function:** anything a Clay formula does in one line (regex, string split, `IF`). Anything that's really just "Claygent with a prompt" — that belongs in a prompt library, not a Function, because wrapping it hides the prompt from the person who actually needs to tune it. And absolutely nothing whose only job is to rename columns.

Credit math check: a Function at $0.05/call × 10k rows × a weekly refresh is $26k/year for *one column*. If the logic could have been a formula, you just invoiced yourself for elegance. Promotion has to clear that bar.

## Q2 — Scope & composition

The engineer's instinct is to decompose into `check_https` + `check_is_parked` + `normalize_domain` and compose upward. From the consumer seat, I will tell you exactly what happens: every layer of composition is a layer of credit cost I can't see and a layer of error surface I can't debug.

**Runtime cost.** If `validate_domain` composes three sub-Functions and Clay bills per Function invocation (it does, in effect, via the underlying calls), a 10k-row run just became 40k calls. At $0.02 blended, that's $800 per refresh instead of $200. Nobody in the audit trail will know why.

**Debug cost is worse.** When a row returns `is_valid: false` and I need to know *why*, tracing through three nested Functions in the Clay UI is miserable. The sandbox shows you one layer. You click into the next. You lose context. By layer three you're copying row IDs into a notepad.

**Where composition earns its keep:** when primitives are *independently useful*. `normalize_domain` is worth standalone status because I want it in dozens of places without dragging validation logic along. `check_is_parked` alone? Nobody calls that in isolation. Bundle it.

**My rule:** compose at most one level deep. A Function can call primitives; primitives don't call primitives. If you need three layers, you're building a framework, not a Function, and you should do that in N8N where you have real observability.

Default to the **single deep Function** for v1. Extract primitives *after* you've seen them get reused, not in anticipation. This is where I disagree with the engineer POV hardest: premature composition in Clay is more expensive than premature coupling, because the credit meter runs whether you refactor later or not.

## Q3 — Interface design

**Inputs:** one primary, plus the minimum to disambiguate. For a domain validator: one `domain_or_url` string. Don't ask the consumer to pre-normalize — that defeats the point. Optional overrides (timeout, strictness) only if I've seen two real consumers ask for different behavior. Config-by-accretion is how Functions become unusable.

**Outputs — this is where Functions live or die.** The Clay UI rewards flat outputs. The moment a consumer has to write `{{function_output.validation.checks.https.passed}}` in a formula, adoption dies silently. Flat, scalar, top-level:

- `is_valid` (bool) — the one field 80% of consumers read
- `normalized_domain` (string) — **yes, emit it.** This is the answer to "what does Claygent need downstream?" Claygent needs a clean URL to visit. If every consumer has to re-normalize after calling the Function, you've failed.
- `failure_reason` (enum, not free text) — `parked | dns_fail | http_error | invalid_format | ok`. Enums are consumable in formulas and scoring. Free-text reasoning is decoration.

**Reasoning strings:** I will say it plainly — nobody reads them. I have watched `explanation` fields go unused in every scoring Function I've shipped. Include a `debug_notes` field if you must, but don't pretend it's part of the interface. It's for the author, reading logs at 11pm.

**Confidence scores:** theater 90% of the time. A 0.73 confidence from a rule-based domain check is made up. Emit confidence only when it comes from a real probabilistic source (model logprobs, multi-signal voting). Otherwise use enums — `high | medium | low` — which consumers can actually branch on in a scoring Function without pretending 0.73 > 0.71 means something.

## Q4 — Versioning contract

From the consumer's seat, the question isn't "how do you version" — it's "how do I find out I'm on a stale version, and how painful is upgrading."

**Breaking vs additive, practitioner definition:**

- **Additive (safe):** new optional output field. New optional input with a default. That's it.
- **Breaking:** renamed field, removed field, changed type, changed enum values, changed default behavior of an existing input. *Especially* the last one — silent behavior changes are the worst kind of break because nothing errors, the numbers just shift.

**Discovery.** Clay doesn't push notifications for Function updates. So the repo needs: a `CHANGELOG.md` per Function, and a **pinned version in every consuming table's Function column**. If the column pins to v1, it stays on v1 forever until I explicitly bump. No auto-upgrade. Ever. I've been burned by "transparent improvements" that retroactively changed 40k rows of scoring.

**Migration.** v2 ships alongside v1 for at least 90 days. The v1 Function's description gets a "SUPERSEDED BY v2 — [link], migration notes: [link]" banner. Migration notes say exactly which output fields changed and give me the formula rewrite.

**When staying on v1 is right:** when the downstream scoring model was trained/calibrated against v1 outputs. Upgrading mid-cycle invalidates comparisons. Also: when v2's "improvement" is a credit-cost increase I don't need.

## Q5 — Documentation format

At the moment of consumption I need, in this order, visible without scrolling: **one-line what it does**, **credit cost per call** (real number, not "varies"), **copy-pasteable example input → example output**, **the three failure modes I'll actually hit**, and **what to pipe the output into next** ("feed `normalized_domain` to Claygent's URL field"). Spec-style docs — parameter tables, type definitions, architectural diagrams — I read once and never again. The example block is the doc.

## Q6 — Failure modes

**Credit blowups.** The classic: someone drags a Function onto a 50k-row table without realizing it calls a paid API per row. $4k gone before lunch. **Guardrail:** every Function's description has the per-call cost *in the name or first line*. `validate_domain ($0.02/call)`. Make it impossible to not see.

**Silent Claygent failures.** Claygent returns "I couldn't access the page" as a successful response. The Function wrapping it returns `is_valid: true` with garbage. **Guardrail:** any Function consuming Claygent output must check for the known failure phrases ("privacy error," "I was unable to," "couldn't access") and map them to an explicit `failure_reason: claygent_blocked`. Not optional. I have lost days to this.

**Orphan Functions.** Function gets built, used in two tables, original author leaves, tables get archived, Function sits there. Six months later someone builds a near-duplicate because search didn't surface it. **Guardrail:** quarterly audit — Functions with zero calls in 90 days get a deprecation banner. Two quarters idle, they're archived. The repo owns this cadence.

**Nested-Function debugging hellholes.** Covered in Q2 but worth repeating: a failing row in a composed Function is a 20-minute investigation minimum. **Guardrail:** every Function emits a `trace_id` or at least a `debug_notes` scalar that names which internal step failed. Not a stack trace — just "failed at: dns_lookup". Cheap to emit, saves hours.

**The waterfall overlap trap.** Team A builds `validate_domain`. Team B builds `check_company_domain` which does 80% of the same thing. Both run on the same 10k table. That's $1600 to answer the same question twice. **Guardrail:** the repo maintains a one-page "what exists" index, and new-Function PRs require a "why not extend X" section. Not bureaucracy — it's the one thing that actually prevents duplicate spend.

**The consumption-ergonomics trap.** Function works perfectly, nobody uses it because the output is nested three deep and the formula to consume it is illegible. Dies quietly. **Guardrail:** Q3's flat-output rule, enforced in review. If the example in the doc needs a multi-line formula to unwrap, the interface is wrong.
