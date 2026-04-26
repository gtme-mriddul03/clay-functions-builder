---
name: clay-functions-v2
description: Design, version, document, and govern Clay Functions (reusable enrichment workflows in Clay). Use whenever the user is working with Clay Functions — designing or scoping one, cutting a new version, writing or reviewing the spec/usage doc, preparing to publish, logging a design decision, or debugging interface drift across tables. Also triggers when the user is *about to* build one and needs structure first, or when they describe symptoms of missing structure ("my Clay column broke after I renamed an output", "I keep copy-pasting this enrichment across tables", "I want to standardize my Clay workflow", "should this be one function or several", "how do I add a check without breaking my existing tables"). Skill triggers on: "Clay Function", "Clay enrichment", "should I build a function", "function scope/design/interface/output/input/version", "breaking change", "additive change", "function spec", "function canvas", "function ADR", "publish Clay Function", "validate domain function", "compose Clay Functions", "Clay column broke after rename".
---

# Clay Functions Skill

This skill governs how we design, version, document, and publish Clay Functions — reusable enrichment workflows with defined inputs, a workflow body, and structured outputs, referenced as a single column in any Clay table.

Clay handles internal workflow diffs safely via sandboxed editing. What Clay does **not** enforce is what counts as a breaking vs additive interface change. We enforce that. This skill is that policy.

## When to use which file

| Task | Go to |
|---|---|
| Should I build a Function at all? | [sop/00-when-to-build.md](sop/00-when-to-build.md) |
| Designing scope and interface | [sop/01-design.md](sop/01-design.md) |
| Cutting a new version | [sop/02-version.md](sop/02-version.md) |
| Writing documentation | [sop/03-document.md](sop/03-document.md) |
| About to publish | [sop/04-pre-publish-checklist.md](sop/04-pre-publish-checklist.md) |
| Logging a design decision | [references/adr-guide.md](references/adr-guide.md) |
| Understanding how Functions fail and how the framework guards against each | [references/failure-modes.md](references/failure-modes.md) |
| Blank templates | [assets/templates/](assets/templates/) |
| Mental model / why this exists | [references/mental-model.md](references/mental-model.md) |

## Start here if you're new

Read `references/mental-model.md` first — it's short. Then run through `sop/00-when-to-build.md` on your first candidate Function. The templates in `assets/templates/` are fillable starting points; they're not worth reading cold.

The `docs/` and `working/` directories ship empty by design — they hold per-Function output (filled specs, filled usage docs) and pre-build canvas drafts respectively, populated as you use the skill rather than pre-loaded with content.

## The opinionated positions this skill takes

These are decisions, not suggestions. If you want to revisit any of them, log an ADR.

1. **Build gate**: a pattern earns a Function when it's used in 2+ tables, OR a confirmed second use is incoming this week. Not before. Full gate in [`sop/00`](sop/00-when-to-build.md).
2. **Composition default**: start with one deep Function. Extract a primitive only when it is concretely needed by a *different* Function (not just another table). Full rule in [`sop/01`](sop/01-design.md#step-4--composition-check).
3. **Outputs**: flat by default. One nesting level is OK for a coherent sub-object with 3+ related fields. No deeper. Full rule in [`sop/01`](sop/01-design.md#step-3--design-outputs).
4. **Reasoning strings**: include one only if a downstream consumer *branches on* the reasoning text (the next step reads its content and acts differently). Omit otherwise.
5. **Confidence scores**: never include unless the consuming workflow has an explicit threshold gate that reads the score.
6. **Versions**: removing/renaming an output field, changing a field's type, removing a required input, or changing the default of an existing optional input in a way that affects current callers — all breaking, all require a new version. Adding a new optional output field or a new optional input with a behavior-preserving default is additive (no bump). Full classification in [`sop/02`](sop/02-version.md#step-1--classify-the-change).
7. **Deprecation**: old versions stay live for 30 days after the new version ships. After 30 days, remove with no ceremony.
