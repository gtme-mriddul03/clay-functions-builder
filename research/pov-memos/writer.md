# Writer POV — memo

## Q1 — Build gates

A pattern earns Function status when it's been copy-pasted three times across tables and at least one of those copies drifted. Not before. "We might reuse this" is not a gate — it's a wish. Things that should NOT be Functions: one-off cleanups, table-specific business logic, anything whose inputs you can't name in under ten seconds.

The gate itself has to be a **five-line checklist at the top of the canvas**, not a paragraph buried in a governance doc. Literal format:

```
[ ] Used 3+ times
[ ] Inputs fit on one line
[ ] Output shape is stable
[ ] Owner named
[ ] Not blocked by a vendor quirk
```

If a tired person at 11pm can't tick all five, it's not a Function yet. The Engineer POV will want nuance here — resist it. Nuance in a gate is how you get Functions-by-vibes. A checklist that fits in a screenshot survives; a rubric with weighted criteria does not.

## Q2 — Scope & composition

Split when the concern has its own failure mode, its own output shape, or its own vendor. That's it. Those three tests, in that order.

Domain validator illustrates: "is this a real domain" and "what's the canonical root" share a failure mode (DNS/WHOIS) and an output shape (a domain string + metadata) — one Function. "Is this domain a competitor" has a different failure mode (list lookup) and a different vendor — its own Function, composed.

The Engineer POV will argue for primitives-first on principle. I disagree for the human reason: every extra Function is another library entry, another usage doc, another version to track. Composition is free at runtime and expensive at documentation time. Default to one Function with a `depth` option; split only when the doc for the combined thing exceeds one screen. The doc length is the test.

## Q3 — Interface design

Required inputs: the thing without which the Function is meaningless. Usually one, sometimes two. If you have three required inputs, you have two Functions pretending to be one.

Optional inputs earn their place by answering a yes/no question the caller actually asks — `include_subdomains`, `depth: "shallow" | "deep"`. Banned: optional inputs that toggle internal implementation. The caller doesn't care which API you hit.

Outputs: start minimal, add fields only when a consumer asks twice. The domain validator's `reasoning` string is the interesting case — I'd ship it in v1 **only** because a validator without a reason is a black box that ops teams stop trusting by week three. That's an interface-carries-the-doc decision: a good `reasoning` string means the usage doc doesn't need a "why did it say no" section.

`confidence` fields are earned when someone has filtered on them. `debug` fields are earned never — use logs. Self-documenting interfaces have output keys that read like English sentences (`is_valid_domain`, `resolved_root`, `reason`). When you need a legend to read the output, docs are now carrying weight they shouldn't be.

The rule: if the output field names are good, the usage doc is three paragraphs. If they're cryptic, it's three pages. Pay the naming tax up front.

## Q4 — Versioning contract

Additive: new optional input, new output field, performance change. Safe to ship, mention in a changelog line.

Breaking: removed output field, renamed field, changed type, changed semantics of an existing field (this is the sneaky one — same name, different meaning, is the worst kind of break). Also breaking: tightening an input validation that previously passed.

Cut v2 when you have **two or more** breaking changes queued. One breaking change alone is usually a sign you got v1 wrong and should have thought harder — but ship it, don't accumulate. Never run v1 and v2 in parallel for more than 30 days. Parallel versions are how you end up with six live versions and nobody knowing which is canonical.

Communication: the version bump announcement lives in **three places, in this order of authority**:

1. The Function's library entry (source of truth, dated).
2. A pinned changelog row at the top of the spec.
3. One Slack message with a link to #1.

Slack is not documentation. Slack is a notification that documentation changed. If the library entry doesn't say it, it didn't happen. Consumers who don't follow Slack should be able to diff the library entry and see what changed — which means the library entry needs a visible "last changed" date and a one-line "what changed" field. Not a full migration guide in Slack. A link.

## Q5 — Documentation format

Four artifacts, each with a defined job. Mixing jobs is how frameworks die.

**Canvas — one screen, Q1–Q6, period.** This is the pre-build thinking document. ~250-400 words total. Six H2 headers, one paragraph each, no sub-bullets deeper than one level. Banned from canvas: code, output schemas, vendor discussion, "alternatives considered." If it doesn't fit on one screen at normal zoom, it's not a canvas, it's a design doc — and nobody reads design docs. The canvas is the thing you fill out *before* you build. Fifteen minutes. If it takes an hour, the template is wrong.

**Spec — ≤120 lines, structured.** This is the contract. Fixed sections in fixed order: Purpose (2 sentences), Inputs (table), Outputs (table), Errors (table), Version (one line), Owner (one line), Changelog (reverse chrono, one line per entry). Banned from spec: prose explanations, examples, rationale. The spec answers "what does this thing do" — not "why" and not "how to use it." If you need more than 120 lines, your Function is too big. The line limit is the pressure that keeps scope honest.

**Library entry — the card, not the book.** One row in a table: name, one-sentence purpose, owner, version, last changed, link to spec, link to usage doc. That's it. The library is a navigation surface, not a content surface. Seven columns max. A new hire should be able to scan the library in under two minutes and know what exists.

**Usage doc — three questions in this order.** (1) What do I pass in? (2) What do I get back? (3) What are the three most common mistakes? That's the whole template. Maybe 150-300 words plus one worked example. Banned: architecture diagrams, vendor rationale, version history (that lives in the spec). The usage doc is written for the person who just opened their Clay table at 11pm and needs this Function to work in the next ten minutes.

**Checklist vs narrative.** Checklist for: build gates, pre-publish review, common mistakes. Narrative for: purpose sentences, reasoning explanations, the worked example in the usage doc. Rule: if the reader is *deciding*, give them a checklist. If they're *understanding*, give them two sentences. Never both for the same job — a checklist with narrative next to it means nobody trusts either.

The Practitioner POV will want more room in the usage doc for edge cases. No. Edge cases go in the spec's error table, one line each. The usage doc stays short or it stops getting read.

## Q6 — Failure modes

Documentation frameworks die of four things, in this order of frequency:

**1. Drift.** Same fact in three places — two will go stale within a quarter. The fix is single-sourcing: spec is canonical for interface, library entry is canonical for metadata, usage doc is canonical for examples. If you find yourself writing the input list twice, stop.

**2. Template bloat.** Every post-mortem adds a new section to the template. After a year the canvas has sixteen fields and nobody fills it. Defend the template aggressively — adding a section requires removing one. No exceptions.

**3. TL;DRs nobody updates.** A summary at the top of a doc that drifts from the body is worse than no summary. Ban TL;DRs in specs. Allow them only in usage docs, and only if they're auto-derivable from the first question ("what do I pass in").

**4. Abandonment.** The second Function is never documented as well as the first. Fix: the gate in Q1 includes "owner named." No owner, no Function. The owner's job isn't to maintain forever — it's to either maintain or formally hand off. Unowned Functions get archived after 90 days of inactivity. Make archiving routine, not dramatic. A library of 12 live Functions beats a library of 40 where half are zombies.

The Skeptic POV will say all of this is overhead. Partly right — which is why every artifact here has a line limit. Overhead that fits on one screen gets used. Overhead that spans three docs gets abandoned by month four.
