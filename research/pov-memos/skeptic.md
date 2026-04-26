# Skeptic POV — memo

## Q1 — Build gates

Most Functions shouldn't exist. Before anyone opens the builder, they should have to answer — on paper, in the PR description, not in their head — the following, and fail any one of them kills the proposal:

1. **Second-caller test.** Name the second table that will reference this Function. Not "could" — *will*, this quarter, with a table URL or a Jira ticket. A Function with one caller is a workflow step with extra governance overhead. If the author can't name caller #2, it's a table step. Build it as a table step. Promote later if the demand actually materializes. It won't, 70% of the time.
2. **Formula test.** If the body is a single Claygent call, a single HTTP call, or a regex-and-lowercase, it's not a Function, it's a formula with a vanity wrapper. The Engineer POV will tell you "wrap it anyway, for consistency." Ignore them. The wrapper adds a version number, a deprecation clock, a credit line item, and a maintainer obligation, in exchange for saving three seconds of typing. That's a bad trade 100% of the time.
3. **Maintainer-on-the-hook test.** A named human — not "the RevOps team," a human with an email — must commit to owning it for 12 months. If that human leaves, the Function goes on a 60-day deprecation clock automatically. No exceptions. The "orphaned load-bearing Function" is the single most common failure mode I see in mature workspaces, and it's always because nobody signed on the line when it was born.
4. **Kill criteria, pre-stated.** What metric, at what threshold, triggers deprecation? "Fewer than 3 callers after 90 days" is a real answer. "We'll see how it goes" is not.

The Practitioner POV will call these gates "bureaucratic." Good. Bureaucracy is what keeps the Functions library from becoming the 400-item Zapier graveyard every growth team has in a drawer.

## Q2 — Scope & composition

"Compose primitives" is the seductive lie of this repo. In theory: a `normalize_url` primitive, a `fetch_whois` primitive, a `classify_domain_type` primitive, and your domain validator is a four-line orchestration. In practice: six months in, `normalize_url` has a v1 used by marketing (strips `www.`) and a v2 used by sales (keeps `www.` because the CRM matches on exact string), and the "validator" has a branching input flag to pick which normalizer, and now you have three Functions where one would do.

Composition collapses the moment two callers disagree on a primitive's behavior. And they always disagree, because the primitive's author didn't know about caller #2 when they wrote it — see Q1.

A "primitive" is just a formula pretending to be a Function when: (a) its body is one call, (b) its only output is the raw result of that call, and (c) no caller composes it with a *different* primitive inside another Function. Condition (c) is the real test. If `normalize_url` is only ever called standalone from a table column, it's not a primitive, it's a utility, and it should live as a formula snippet in docs, not a versioned Function.

**Guardrail:** a Function may only be classified "primitive" after two *Functions* (not tables) call it. Until then, it's a candidate, and candidates don't get v-numbers.

## Q3 — Interface design

Output fields I have watched rot, in order of frequency:

- **`reasoning` / `explanation` strings.** Read by zero humans after week two. Parsed by zero downstream Functions, ever. They exist because the author felt good including them. They cost tokens on every run, they bloat the column, and when the LLM changes its phrasing the "reasoning" drifts and nobody notices because nobody reads it.
- **`confidence_score` floats.** Used by exactly one caller, with a threshold picked by vibes. Never recalibrated. When the underlying model changes, the distribution shifts and the threshold silently stops meaning what it used to.
- **`alternates` / `candidates` arrays.** Populated out of "future-proofing," consumed never. Downstream Functions always take `[0]`. Delete the array, expose the winner.
- **`raw_response` / `debug` blobs.** Swell 10x over the useful output. Authors swear they'll remove them "after launch." They don't.

**Load-bearing test, applied before publish:** for each output field, name the downstream consumer *by column or Function*. Not "someone might want this." A column. If you can't, the field is theater. Cut it. You can always add a field in a minor version — see Q4 — but you cannot remove one.

The Writer POV will say "include reasoning for observability." Log it, don't output it. Different concerns.

## Q4 — Versioning contract

Every versioning policy I've seen break has broken the same way: the team defined "breaking" by *shape* (added/removed/renamed fields) and ignored *semantics*. Then someone changed what `resolved_domain` meant — from "the registered root" to "the canonical hostname" — without renaming the field, shipped it as v1.3, and broke every caller silently. Schema diff was clean. The data was different. No alarm fired.

Second failure: deprecation windows nobody honors. v1 is deprecated at publish of v2, with a "60-day sunset." Day 61, v1 still has eleven callers, none of whom got paged. The sunset becomes "we'll get to it." Two years later v1 is the *more* popular version because its quirks are depended on.

Third: naming collisions. `domain_validator`, `domain_validator_v2`, `domain_validator_new`, `domain_validator_final`, `domain_validator_jake`. All alive. Nobody knows which is canonical.

**Guardrails that actually work:**
- Any change to *meaning* of an existing field is a new Function name, not a new version. Versions are for adding fields and fixing bugs within fixed semantics. Period.
- Deprecation is enforced by tooling, not culture: at sunset, the old version's run errors with a migration link. If that's too aggressive for your culture, your culture is the problem, not the policy.
- One name per concept, ever. No `_v2` in the name. No `_new`. No author initials.

The Engineer POV wants semver. Fine — but only if someone owns the registry and pages owners when sunsets hit. Otherwise semver is just three-numbered decoration.

## Q5 — Documentation format

What gets read: the one-line purpose at the top, the input/output table, and one real example with real values. Everything below the fold is written for the author, not the reader.

What dies: the "Architecture" section, the "Design rationale" section, the "Known limitations" section (which becomes a lies-told-to-future-maintainers section within two quarters as limitations are quietly fixed or forgotten), and any canvas with more than four boxes. Mermaid diagrams of Function composition are written once, rendered zero times after the PR merges, and wrong within a month.

**Keep:** purpose line, I/O table, one worked example with actual input and actual output values, owner + sunset date, link to the two nearest-neighbor Functions (so the reader can tell if they're in the wrong doc). That's it.

**Kill:** anything that restates the builder UI, any "philosophy" section, any template field the author left as `TODO`.

## Q6 — Failure modes

Six failure modes I have watched play out, each with the one guardrail that would have stopped it:

1. **Semantic drift.** `resolved_domain` changes meaning between v1.2 and v1.3. No schema diff, no alarm. Callers silently break. **Guardrail:** every output field gets a pinned example value in the spec; CI fails the publish if the example no longer matches the current output for the same input. Locks semantics to a test, not a sentence.

2. **Orphaned load-bearing Functions.** Author leaves, Function keeps running, nobody touches it, it gradually rots as upstream APIs change. **Guardrail:** ownership expires automatically at 12 months. The Function enters a 60-day "needs new owner or dies" state. Forces either renewal or honest death.

3. **Workspace-scope creep.** Marketing's `score_account` gets called by sales, who need a different weighting, so marketing adds a `mode` flag, then CS adds another, and now the Function serves nobody well. **Guardrail:** a Function that adds a `mode` / `variant` / `team` input parameter is forked, not extended. Three callers with three modes is three Functions. The appearance of a switch parameter is a code smell, full stop.

4. **Credit-cost surprises.** Nested Functions fan out — validator calls enricher calls classifier — and a 10,000-row run racks up a $4k bill nobody forecasted. **Guardrail:** published Functions must declare a per-call credit ceiling; the builder fails to save if a composition exceeds caller's declared ceiling. Make cost a type-checked thing.

5. **Nested debugging hell.** A Function three layers deep returns garbage. The user sees only the top-level output. Tracing takes an afternoon. **Guardrail:** cap composition depth at 2. A Function may call Functions, but those Functions may not call Functions. If your design needs depth 3, your design is wrong, or you need a new primitive at depth 1.

6. **"Reasoning string" graveyard.** Every Function ships with a reasoning output, none are read, token costs compound. **Guardrail:** output fields without a named downstream consumer in the spec are rejected at publish. Observability goes to logs, not to the column.

None of these are hypothetical. All of them are in every mature Clay workspace I have audited. This repo's job is to make them harder to repeat than to repeat.
