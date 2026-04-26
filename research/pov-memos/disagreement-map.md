# Disagreement Map

Synthesized from `engineer.md`, `writer.md`, `practitioner.md`, `skeptic.md`.

Ordered **load-bearing first**. A disagreement is load-bearing if a downstream SOP or template depends on its resolution; peripheral if it's flavor that can be settled later without cascading. Resolutions are defaults for Mriddul to approve or override inline.

---

## Load-bearing

### A. Compose primitives first vs ship a single deep Function first

- **What:** When building a new concern that could be expressed as either one Function or several composed ones, which is the default?
- **Positions:**
  - **Engineer:** primitives-first. Extract `check_https`, `check_is_parked`, `normalize_domain` up front; `validate_domain` composes them. Shipping only the composed version traps future reuse.
  - **Practitioner:** single deep Function first for v1. Extracting primitives in anticipation burns credits and debugging time. Promote to primitive *after* observed reuse.
  - **Writer:** agrees with Practitioner on pragmatic grounds — "composition is free at runtime and expensive at documentation time."
  - **Skeptic:** sides with Practitioner by implication: "primitive" status is earned only after two *Function* callers exist, not handed out prospectively.
- **Proposed resolution:** **Practitioner default wins.** Ship as one Function. Primitives are extracted only when (a) a second Function (not a table) wants the sub-step, or (b) the sub-step is independently useful in isolation and you can name a table that would consume it today. Justification: 3-of-4 POVs, and it matches the observed failure mode (premature primitives orphan or fragment).
- **Load-bearing:** yes — drives [sop/01-design.md](../../skill/sop/01-design.md) and the function-canvas template.

### B. `depth` / `mode` toggle inside one Function — default pattern or anti-pattern?

- **What:** Can a single Function expose a `depth: shallow | deep` or `mode: marketing | sales` input to collapse variants into one contract?
- **Positions:**
  - **Engineer:** anti-pattern. "Two Functions pretending to be one, with a shared output schema that's a lowest-common-denominator lie."
  - **Writer:** explicitly endorsed — "default to one Function with a `depth` option; split only when the doc for the combined thing exceeds one screen."
  - **Practitioner:** silent on the toggle itself; defaults to single-deep Function, which is adjacent but not identical.
  - **Skeptic:** hardest line — "a Function that adds a `mode` / `variant` / `team` input parameter is forked, not extended. The appearance of a switch parameter is a code smell, full stop."
- **Proposed resolution:** **Ban `mode` / `variant` / `team` switches outright** (Skeptic+Engineer). Allow `depth`-style *quantitative* toggles only when (a) all depths share the exact same output schema and field semantics, and (b) a deeper depth is a strict superset of a shallower one. This carves out Writer's legitimate use (cheap/expensive variants of the same answer) while blocking the branching-behavior smell.
- **Load-bearing:** yes — directly drives the domain-validator design call and interface rules in [sop/01-design.md](../../skill/sop/01-design.md).

### C. `reasoning` string output — ship it, log it, or kill it?

- **What:** Should a composed/judgment Function include a human-readable `reasoning` or `explanation` field in its output?
- **Positions:**
  - **Engineer:** ship by default, human-readable, explicitly never parsed downstream. "I'd rather ship one extra boolean than force a v2 in four months."
  - **Writer:** ship in v1 *because* a good reasoning string lets the usage doc stay short ("interface carries the doc").
  - **Practitioner:** nobody reads them. If you must, call it `debug_notes` and don't pretend it's part of the interface.
  - **Skeptic:** theater. "Log reasoning, don't output it. Different concerns."
- **Proposed resolution:** **Split the difference.** Ship a short (`≤200 char`) `reasoning` string only for composed/judgment Functions *and* only when the output includes a verdict field (`is_valid`, `classification`, etc.). Primitives never get one. Every Function with a `reasoning` field must declare it "human-only, not for machine parse" in the spec, and the spec's load-bearing-field check (Skeptic's guardrail, see J below) does not apply to it. Justification: respects Writer's doc-saving use case and Engineer's debugging use case while neutering Skeptic/Practitioner's "rots silently" objection by making it honest about its role.
- **Load-bearing:** yes — [templates/function-spec.md](../../skill/assets/templates/function-spec.md), [sop/01-design.md](../../skill/sop/01-design.md), [sop/03-document.md](../../skill/sop/03-document.md).

### D. `confidence_score` fields — keep, replace with enum, or kill?

- **What:** Do Functions emit numerical confidence scores?
- **Positions:**
  - **Engineer:** skip by default ("you don't have calibrated ones"), add when asked.
  - **Writer:** earn the field only when a downstream consumer actually filters on it.
  - **Practitioner:** theater 90% of the time unless from a probabilistic source. Use `high | medium | low` enum instead, which consumers can branch on without pretending 0.73 > 0.71 means something.
  - **Skeptic:** cut. Fields that rot with a threshold-by-vibes.
- **Proposed resolution:** **Default: no confidence field.** If the author insists, two conditions must hold: (1) the score comes from a real probabilistic source (model logprobs, multi-signal vote), and (2) at least one named downstream consumer filters on it. Otherwise, emit Practitioner's enum (`high | medium | low`) — consumable in formulas, no false precision. Justification: Practitioner's enum is the only form all four POVs tolerate.
- **Load-bearing:** yes — interface policy in [sop/01-design.md](../../skill/sop/01-design.md).

### E. Version naming — `_v2` suffix vs one-name-per-concept

- **What:** When v2 ships, does its name carry the version (`validate_domain_v2`) or is there exactly one canonical name with versions tracked out-of-band?
- **Positions:**
  - **Engineer:** `_v2` suffix. "Ugly, load-bearing, non-negotiable."
  - **Skeptic:** "One name per concept, ever. No `_v2` in the name. No `_new`. No author initials." Versions are for bug fixes inside fixed semantics; anything else is a new name *for a new concept*.
  - **Practitioner:** silent on naming; pin-per-column implies some distinguishable handle exists.
  - **Writer:** silent on convention; library entry tracks `version` + `last changed` date.
- **Proposed resolution:** **Engineer's `_v2` suffix wins** for breaking changes, because Clay provides no native version namespace and consumers must distinguish them in the column picker. BUT the Skeptic-style rule applies to *additive* changes within a major: those are tracked as a `version:` field in the spec (e.g., `1.1`, `1.2`) without a name change. Net effect: `validate_domain` (stable, v1.x) and `validate_domain_v2` (stable, v2.x) coexist. No `_new`, no `_final`, no initials. Justification: honors Clay's lack of namespacing while preserving Skeptic's anti-sprawl intent.
- **Load-bearing:** yes — [sop/02-version.md](../../skill/sop/02-version.md).

### F. Deprecation enforcement — manual registry, consumer-pinned, or hard tooling cut?

- **What:** How is v1 actually retired once v2 ships?
- **Positions:**
  - **Engineer:** manual registry of consumers; don't delete v1 until registry shows zero callers, regardless of sunset date.
  - **Practitioner:** no auto-upgrade, ever. Consumers pin per column. They migrate when *they* are ready; v2's existence is a notification, not a forcing function.
  - **Skeptic:** tooling-enforced hard cut at sunset. "If that's too aggressive for your culture, your culture is the problem."
  - **Writer:** no parallel versions beyond 30 days — implicit forcing function.
- **Proposed resolution:** **Practitioner's pin-per-column as the default, Engineer's registry as the audit trail, Skeptic's hard cut reserved for security/correctness bumps only.** Routine deprecation: v1 carries a `deprecated` banner and a sunset date; the registry tracks consumers; migrations happen on the consumer's schedule. Security or correctness-critical v2s (the v1 is returning *wrong* data) trigger the Skeptic hard-cut path: registry → owner pages → hard removal at sunset. Writer's 30-day rule is rejected — it creates panic migrations. Justification: consumers own their data pipeline's stability; the framework's job is visibility, not coercion (except when v1 is actively harmful).
- **Load-bearing:** yes — [sop/02-version.md](../../skill/sop/02-version.md).

### G. Deprecation window length — 30, 60, or 90 days?

- **What:** Minimum time v1 runs in parallel with v2 before retirement.
- **Positions:**
  - **Engineer:** 90 days minimum.
  - **Practitioner:** 90 days minimum.
  - **Skeptic:** 60 days then hard cut.
  - **Writer:** 30 days max — wants a tighter window to prevent version proliferation.
- **Proposed resolution:** **90 days default, 60 days for the security/correctness hard-cut path (resolution F).** Writer's 30-day ceiling is rejected as incompatible with the pin-per-column model — a 30-day forced migration *is* the panic migration F was designed to avoid. Justification: 3-of-4 POVs land at ≥60; 90 matches Engineer+Practitioner's threshold and the quarterly audit cadence.
- **Load-bearing:** yes — [sop/02-version.md](../../skill/sop/02-version.md), [sop/04-pre-publish-checklist.md](../../skill/sop/04-pre-publish-checklist.md).

### H. Composition depth cap — 1 or 2 levels?

- **What:** Can a composed Function call another composed Function, or only primitives?
- **Positions:**
  - **Engineer:** 2 levels (primitive → composed). Composed-of-composed is a debugging nightmare.
  - **Practitioner:** 1 level ("primitives don't call primitives"). Anything deeper is N8N territory.
  - **Skeptic:** 2 levels, stated as "a Function may call Functions, but those Functions may not call Functions."
  - **Writer:** silent.
- **Proposed resolution:** **Cap at 2 levels** (Engineer+Skeptic). Justification: a hard 1-level cap would prohibit a composed `validate_domain` from internally using a primitive that itself wraps one vendor call — Practitioner's real concern is debuggability, which the framework addresses separately via trace_id emission (Practitioner's own Q6 guardrail). Two levels gives headroom for genuine primitive reuse without opening the depth-3 abyss.
- **Load-bearing:** yes — [sop/01-design.md](../../skill/sop/01-design.md).

### I. Build-gate threshold — count-based vs named-second-caller test

- **What:** Does promotion require N observed uses, or a named second caller?
- **Positions:**
  - **Engineer:** 3+ tables OR workflow with other human editors + stable shape + non-trivial body.
  - **Practitioner:** ≥3 tables OR ≥2 workspaces + stable inputs + deterministic-to-cache.
  - **Writer:** 3 copy-pastes with at least one drift. Five-line checklist at the top of the canvas.
  - **Skeptic:** name caller #2 *by URL or ticket*, this quarter. "Will," not "could."
- **Proposed resolution:** **Writer's five-line checklist as the surface, Skeptic's second-caller test as one of the five lines.** The full gate:
  1. Used or committed in 3+ tables OR used in a workflow other humans edit
  2. Caller #2 is nameable now (table URL, ticket, or "I'm building it this week")
  3. Inputs fit on one line; output shape is stable
  4. Body is non-trivial (not formula-able, not a single Claygent call)
  5. Owner named, with a 12-month commitment
  
  Justification: Writer's format makes the gate survive, Skeptic's named-caller rule makes it bite, Engineer/Practitioner's thresholds provide the numeric floor.
- **Load-bearing:** yes — [sop/00-when-to-build.md](../../skill/sop/00-when-to-build.md).

### J. Load-bearing output-field check — mandatory pre-publish or optional?

- **What:** Before publishing, must every output field name a downstream consumer (column or Function)?
- **Positions:**
  - **Skeptic:** mandatory. "Output fields without a named downstream consumer in the spec are rejected at publish." The single strongest guardrail against theater fields.
  - **Engineer:** aligned in spirit (cost budgets, consumer registry) but less rigid — `reasoning` ships by default without a named consumer.
  - **Writer:** aligned on principle ("outputs: start minimal, add fields only when a consumer asks twice") but doesn't make it a publish gate.
  - **Practitioner:** aligned in spirit; wants flat outputs and named consumers.
- **Proposed resolution:** **Adopt Skeptic's rule with one carve-out for `reasoning` (resolution C).** Every output field in the spec must name its downstream consumer (column, Function, or scoring formula) *except* `reasoning`, which is explicitly human-only and must be declared as such. This becomes item 1 on the [sop/04-pre-publish-checklist.md](../../skill/sop/04-pre-publish-checklist.md). Justification: convergence across all four POVs on the principle; Skeptic's framing is the most actionable.
- **Load-bearing:** yes — [sop/04-pre-publish-checklist.md](../../skill/sop/04-pre-publish-checklist.md), [templates/function-spec.md](../../skill/assets/templates/function-spec.md).

---

## Peripheral

### K. Cost visibility — in the Function name or just the spec?

- **What:** Does `validate_domain ($0.02/call)` appear in the Function's display name, or only in the spec?
- **Positions:** Practitioner wants it in the name ("make it impossible to not see"). Engineer/Skeptic want it in the spec with budget enforcement. Writer is silent.
- **Proposed resolution:** **Spec, not name.** Names are unstable enough already (resolution E). Cost goes in the spec's one-line header and in the library entry as a column. Justification: naming-as-communication fights naming-as-identity.
- **Load-bearing:** no — affects [templates/function-spec.md](../../skill/assets/templates/function-spec.md) and library-entry.md (v1-only, not in v2 skill) but doesn't cascade.

### L. Failure signaling — enum vs string vs both

- **What:** How does a Function communicate *why* it returned a negative verdict?
- **Positions:** Practitioner: enum only (`parked | dns_fail | ...`). Engineer: structured booleans + `reasoning` string. Writer: output field names that read like English sentences. Skeptic: kill the string.
- **Proposed resolution:** **Enum (`failure_reason`) is canonical and load-bearing; `reasoning` string is the optional human-facing supplement per resolution C.** Booleans-per-check (Engineer's proposal) are allowed additively when the verdict is composed from named sub-checks. Justification: enums survive, strings drift, booleans scale when a verdict has 2–4 inputs and break when it has 10.
- **Load-bearing:** no — follows from resolutions C and D.

### M. Canvas length target

- **What:** How long is the function-canvas template?
- **Positions:** Writer: one screen, Q1–Q6, ~250–400 words. Skeptic: four boxes max. Engineer conflates canvas with an in-Clay one-line explainer.
- **Proposed resolution:** **Writer's format wins** — one screen, Q1–Q6, ≤400 words, filled pre-build in 15 minutes. The "in-Clay one-line explainer" Engineer describes is actually the *library entry's* purpose field, not the canvas. Clarify this nomenclature in [templates/function-canvas.md](../../skill/assets/templates/function-canvas.md).
- **Load-bearing:** no — template-local.

---

## Notes for synthesis

- **Resolutions C (reasoning string carve-out), F (deprecation enforcement split), and I (gate as combined checklist) are the load-bearing compromises.** If Mriddul overrides any of these, several downstream SOPs rewire.
- **Convergences worth noting** (not disagreements but worth recording): all four POVs treat *semantic drift on a field with a stable schema* as a breaking change; all four require a named owner; all four want a quarterly orphan audit; all four endorse flat scalar outputs; all four want the resolved/canonical form of the input emitted.
- **The domain-validator tension** resolves from the above as: single Function `validate_domain` v1, no `mode` flag, outputs `is_valid` (bool) + `normalized_domain` (string) + `failure_reason` (enum) + `reasoning` (short string, human-only), primitives extracted only when a second Function wants them. This is a worked example, not a decision — Mriddul builds the actual thing.
