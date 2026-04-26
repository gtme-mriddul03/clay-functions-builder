# Mental Model — Clay Functions

Clay Functions are just named, versioned, reusable columns. That's it. They wrap an enrichment workflow — some combination of API calls, Claygent prompts, formula logic, or sub-Function calls — behind a stable interface. Any Clay table that needs the same enrichment just references the Function instead of rebuilding the workflow inline.

The practical upside is consistency and reuse. The practical risk is interface coupling: the moment a second table references your Function, changing its output shape silently breaks that table. Clay won't warn you. You'll find out when a column downstream shows an error — or worse, silently stops enriching.

## Why this framework exists

Clay handles the *internal* change problem well: sandboxed edits, diff-on-publish, you can see what changed before it goes live. What Clay doesn't handle is the *interface contract* problem. It has no concept of breaking vs additive changes, no version history at the interface level, no deprecation flow.

This framework authors that contract. The goal is that every Function you (or a future hire) ship follows the same shape, so:

- You can read any Function's spec and immediately know its inputs, outputs, and version history.
- Adding a new check or field six months in doesn't break tables referencing the old version.
- A new hire can pick up a Function mid-life without asking what changed.

## The composition question

The hardest design call in Clay Functions is: one deep Function or composed primitives?

The pull toward primitives feels intuitive — small, testable, reusable. But primitives create coordination overhead: you need to call them in sequence, pass outputs between them, and now you have N Functions to document and version instead of one. In a Clay table, each additional Function reference is another column, another enrichment run, another place for things to go wrong.

The rule: start with one Function. Extract a primitive when — and only when — the primitive does something a *different* Function also needs. If `validate_domain` and `enrich_contact` both need to normalize a URL, `normalize_domain` earns its existence. If only `validate_domain` uses it, the normalization logic lives inside `validate_domain` and stays there.

## The output design question

Flat outputs win in Clay. A flat object maps directly to Clay columns without custom extraction logic. Nested outputs require formula gymnastics or a second Function to unpack them — and now you've added complexity with no gain.

The only exception: a coherent sub-object with 3+ related fields (e.g., a "resolved" sub-object with `url`, `redirected_from`, `registrable_domain`) earns one level of nesting. Don't nest the nest. Operational rule: see [`sop/01` Step 3](../sop/01-design.md#step-3--design-outputs).

## The reasoning string question

A reasoning string sounds useful. It usually isn't. If downstream consumers are humans reviewing a Clay table, the reasoning reads fine in a cell. If downstream consumers are other Functions or Claygent prompts, the reasoning string costs tokens and usually gets ignored.

Include a reasoning string if — and only if — a downstream consumer branches on it. "Downstream consumer branches on it" means: the next step in the workflow reads the reasoning value and makes a decision based on its content. Not "it might be useful someday." Not "the reviewer might want to know." A concrete branch.

## The versioning question

The version is the interface contract. If a downstream consumer can continue working without any changes after you publish, it's additive. If it needs to update anything — a column reference, a formula, an expected field name — it's breaking.

The practical heuristic:
- **Adding** a new optional output field: additive. Don't bump the version. (Consumers that don't use the new field won't notice.)
- **Removing or renaming** any field: breaking. New version required.
- **Changing** the type or shape of an existing field (e.g., `status` changes from boolean to string): breaking.
- **Changing input defaults** in a way that changes behavior for existing callers: breaking.

When in doubt: is there any table currently referencing this Function that would behave differently after this change without any edits? If yes, it's breaking.
