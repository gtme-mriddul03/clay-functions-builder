---
name: clay-functions-v2
description: Design, version, document, and govern Clay Functions (reusable enrichment workflows in Clay). Use whenever the user is working with Clay Functions — designing or scoping one, cutting a new version, writing or reviewing the spec/usage doc, preparing to publish, logging a design decision, or debugging interface drift across tables. Also triggers when the user is *about to* build one and needs structure first, or when they describe symptoms of missing structure ("my Clay column broke after I renamed an output", "I keep copy-pasting this enrichment across tables", "I want to standardize my Clay workflow", "should this be one function or several", "how do I add a check without breaking my existing tables"). Skill triggers on: "Clay Function", "Clay enrichment", "should I build a function", "function scope/design/interface/output/input/version", "breaking change", "additive change", "function spec", "function canvas", "function ADR", "publish Clay Function", "validate domain function", "compose Clay Functions", "Clay column broke after rename".
---

# Clay Functions Skill

This skill governs how we design, version, document, and publish Clay Functions — reusable enrichment workflows with defined inputs, a workflow body, and structured outputs, referenced as a single column in any Clay table.

Clay handles internal workflow diffs safely via sandboxed editing. What Clay does **not** enforce is what counts as a breaking vs additive interface change. We enforce that. This skill is that policy.

## How to run a session

**Opening:** When invoked for a new function design, open with a single batched set of 2–5 Socratic discovery questions before running any gate. Cover: what the function does, who the callers are, what data callers already have, what should always disqualify, and whether agents are involved. Do not proceed to gates until these are answered.

**Batching:** All clarifying questions within a phase go out as a single numbered list. Answer together, then advance. No single-question round-trips.

**SOP loading:** Load only the SOP section needed for the current phase. Do not read all files upfront.

**File output:** Check for a `functions/` folder in the working root. Create it if missing. Write each function's files to `functions/<verb_noun>/` — never to the skill's global installation directory (files there are invisible to the function register and won't be versioned with the project).

**Agent architecture:** When a function involves LLM agents, infer internet access, parallelism, and pass-through inputs from the design — do not ask the user to spec this. Present the inferred architecture as a summary for quick confirmation.

## When to use which file

| Task | Go to |
|---|---|
| Should I build a Function at all? | [sop/00-when-to-build.md](sop/00-when-to-build.md) |
| Designing scope and interface | [sop/01-design.md](sop/01-design.md) |
| Cutting a new version | [sop/02-version.md](sop/02-version.md) |
| Writing documentation | [sop/03-document.md](sop/03-document.md) |
| About to publish | [sop/04-pre-publish-checklist.md](sop/04-pre-publish-checklist.md) |
| Logging a design decision | [references/adr-guide.md](references/adr-guide.md) |
| Clay column naming | [references/naming-conventions.md](references/naming-conventions.md) |
| How Functions fail and framework guardrails | [references/failure-modes.md](references/failure-modes.md) |
| Blank templates | [assets/templates/](assets/templates/) |
| Mental model / why this exists | [references/mental-model.md](references/mental-model.md) |

## The opinionated positions this skill takes

1. **Build gate**: a pattern earns a Function when it's used in 2+ tables, OR a confirmed second use is incoming this week. Full gate in [`sop/00`](sop/00-when-to-build.md).
2. **Composition default**: start with one deep Function. Extract a primitive only when it is concretely needed by a *different* Function. Full rule in [`sop/01`](sop/01-design.md#step-6--composition-check).
3. **Outputs**: flat by default. One nesting level OK for a coherent sub-object with 3+ related fields. Full rule in [`sop/01`](sop/01-design.md#step-4--design-outputs).
4. **Reasoning strings**: include only if a downstream step branches on its content. Omit otherwise.
5. **Confidence scores**: never include unless a downstream threshold gate reads the score.
6. **Versions**: removing/renaming an output field, changing a field's type, removing a required input, or changing an existing optional input's default in a way that affects callers — all breaking. Adding a new optional output field or optional input with a behavior-preserving default is additive. Full classification in [`sop/02`](sop/02-version.md).
7. **Deprecation**: old versions stay live for 90 days after the new version ships (60 days for security/correctness breaks). After window closes, remove.
8. **Exclusion inputs**: always ask if there are categories that should always disqualify, regardless of positive criteria. Default to adding `excluded_[concept]` as optional input.
9. **Pass-throughs**: for every external fetch inside a Function, add a corresponding optional input so callers can skip the fetch if they already have the data.
10. **Naming**: Clay columns follow `references/naming-conventions.md`. Function names are `verb_noun`. Clay UI names are `verb_noun_vN`.
