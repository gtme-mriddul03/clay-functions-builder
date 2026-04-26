# Engineer POV — memo

## Q1 — Build gates

Promotion to a Function is a contract commitment, not a convenience. The bar is: **used in three or more tables, or once in a workflow that other humans will edit**. Below that, it's a formula or a Claygent cell — cheaper to duplicate than to govern. Copy-paste is free; a published Function costs you a deprecation window for the rest of its life.

Three gates, all must pass:

1. **Stable shape.** You can name the inputs and outputs without hedging. If you're still discovering the output schema, it's not a Function yet — it's a draft. Functions with `output_1`, `result`, `data` fields are unbuilt.
2. **Reuse pressure.** Either observed (three tables) or structural (the logic is load-bearing for a downstream Function). Speculative reuse — "someone might want this" — is not a gate. It's how you get a library of orphans.
3. **Non-trivial body.** A Function that wraps one HTTP call with no parsing is a formula with extra steps. The value of a Function is the *parsing, normalization, and error handling* it encapsulates — not the call itself.

What stays out: one-off transforms (formula), free-text judgment with no structured output (Claygent cell), anything with side effects that writes to a system of record (N8N — Clay Functions should be pure-ish reads). The Writer POV will want more Functions for legibility; I'd rather have fewer, sharper ones.

## Q2 — Scope & composition

**Default to primitives. Compose upward.** A primitive answers exactly one question and returns a shape a human can hold in their head. A composed Function orchestrates primitives and adds a verdict.

The domain validator tension is the canonical case. `check_https`, `check_http`, `check_is_parked`, `normalize_domain` are primitives — each is individually useful in tables that don't care about "validation" at all. `validate_domain` composes them and adds the verdict layer. Shipping only the composed version is a trap: six months later someone wants just the parked-domain check and either duplicates the logic or pulls the whole composed Function and throws 80% of the output away. Both outcomes are bad.

Rule: **if a sub-step is independently nameable and independently useful, it is its own Function.** The "depth option" pattern — one Function with a `mode: shallow | deep` input — is an anti-pattern. It's two Functions pretending to be one, with a shared output schema that's a lowest-common-denominator lie.

Composition depth cap: **two levels in production.** Primitive → composed is fine. Composed → composed → composed is a debugging nightmare where a cost regression three layers down surfaces as "the table got slow." If you need three levels, the middle layer is probably doing too little and should be inlined.

## Q3 — Interface design

**Inputs: required = the thing the Function is about. Optional = knobs with defaults that match the 80% case.** If every caller passes the same value for an "optional" input, it's not optional — it's a default you haven't committed to. Inputs should be parsed, not validated: accept `https://www.foo.com/path` and a bare `foo.com` and normalize internally. Postel's law at the boundary.

**Outputs: rich, flat, and named for consumers.** A single `status` enum with eight values is a worse interface than three booleans plus a canonical form. Consumers filter on booleans; they write switch statements on enums and forget cases when you add the ninth value.

Ship, by default:

- **The canonical/resolved form** of the input (the normalized domain, the resolved URL). Downstream Claygent will want to visit *something* — don't make every consumer re-derive it.
- **Structured booleans** for each check that fed the verdict.
- **A verdict field** when the Function is a composed judgment.
- **`reasoning: string`** — short, human-readable, for debugging in the Clay UI. Not for machine consumption. Never parse reasoning strings downstream; that's a contract you didn't mean to sign.

Skip, by default: confidence scores (you don't have calibrated ones), alternates arrays (YAGNI — add when a second consumer asks), raw API responses (leak — now the upstream vendor's shape is your contract). The Practitioner will push for fewer fields; I'd rather ship one extra boolean than force a v2 in four months.

## Q4 — Versioning contract

**Semver, strictly applied, with names as the version carrier.** Clay doesn't give you package versions — the Function name *is* the version. `validate_domain` is v1. `validate_domain_v2` is v2. Ugly, load-bearing, non-negotiable.

**Additive is free. Breaking requires a new Function.** Adding an output field, adding an optional input with a default, loosening input parsing — all additive, ship into the existing Function. Renaming an output, removing a field, changing an enum value, tightening input validation, changing the *meaning* of an existing field — all breaking, all require `_v2`.

The trap to avoid: **silent semantic drift.** Changing what `is_valid: true` *means* (e.g., now it also requires MX records) is a breaking change even though the schema is identical. Every downstream filter on that boolean just changed behavior. Policy: **if the set of inputs that produce a given output changes, it's breaking.** Ship a new Function.

Deprecation flow:

1. Publish `validate_domain_v2` with the new shape.
2. Mark `validate_domain` as `deprecated` in its spec header with a sunset date *at least 90 days out* and a link to v2.
3. Inventory consuming tables (this requires Clay to surface it, or we maintain a registry — see Q6).
4. Migrate tables. Do not delete v1 until the registry shows zero consumers, regardless of the sunset date. Orphaning a live table is the one thing that is never worth it.

Stability tiers in the spec header: `experimental` (no compat guarantee, expect churn, don't reference from other Functions), `stable` (semver rules apply), `deprecated` (sunset date required). A Function sits in `experimental` until it has two non-author consumers. This is how capability grows for two years without breaking anyone: breaking changes are always new names, and experimental is where you earn the right to be stable.

## Q5 — Documentation format

**Spec (machine-facing, in-repo):** inputs with types and required/optional, outputs with types and semantics, stability tier, version, deprecation pointer, composed-Function dependencies. This is the contract. Terse. No prose.

**Canvas (in-Clay):** what the Function does in one sentence, example input → example output, known limitations. For the person dragging it onto a table.

**Library entry (discovery):** one-line purpose, when to use it, when *not* to use it (the anti-use-case — the Skeptic will want this doubled in size; they're right), link to spec.

**Usage doc (consumer-facing):** composition examples, common patterns, migration notes from prior version.

**Not in any of these:** implementation details, internal API choices, rationale for design tradeoffs. Those go in a changelog or an ADR. Consumers don't need to know you picked DNS-over-HTTPS; they need to know the output shape and when it lies.

## Q6 — Failure modes

Guardrails the framework must encode:

- **Consumer registry.** Every published Function lists its consumers (tables and other Functions). Without this, deprecation is guesswork and orphans are inevitable. If Clay doesn't expose this natively, the framework mandates a manual registry file updated on publish. Yes, it will drift. Audit quarterly.
- **Naming lockdown.** `domain`, `url`, `website`, `company_url` are not synonyms. The framework ships a glossary; Functions must use terms from it. New terms require a glossary PR. This sounds bureaucratic until you've debugged a table where three Functions each meant something different by "domain."
- **Cost budgets per Function.** Every Function spec declares expected cost per row (credits, external API calls). Composed Functions sum their children. A Function that silently grows from 1 credit to 12 credits across versions is a production incident.
- **No nested experimental.** A stable Function cannot depend on an experimental one. Otherwise the stability tier is a lie.
- **Publish diff review.** Every publish must show the diff against the prior version and flag any breaking change per the Q4 policy. If the diff surfaces a rename or a removed field and the Function isn't being renamed to `_v2`, the publish is blocked.
- **Sprawl ceiling.** Quarterly review: any Function with zero consumers for 90 days is archived. Any primitive with only one consumer for 180 days is a candidate for inlining — it didn't earn its keep.

The unifying principle: **make the blast radius of every change visible before it ships.** Everything else is bookkeeping.
