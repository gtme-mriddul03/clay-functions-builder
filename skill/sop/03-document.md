# SOP 03 — Writing Documentation for a Function

Two documents ship with every Function: a **spec** and a **usage doc**. They serve different readers. Don't conflate them.

| Document | Reader | Lives in | Template |
|---|---|---|---|
| Function spec | You and future maintainers | `functions/<function_name>/spec.md` | `assets/templates/function-spec.md` |
| Usage doc | Clay table builders (including future you) | `functions/<function_name>/usage.md` | `assets/templates/usage-doc.md` |
| Backlog | You and future maintainers | `functions/<function_name>/backlog.md` | none — plain list |

---

## When to write each

Write both **before you publish**. Not after. If you can't fill the spec before building, you don't know what you're building yet.

The spec is filled during design (`sop/01-design.md`). The usage doc is filled after you've run the Function once and seen real output.

---

## The spec — what it contains and why

The spec is the source of truth for the Function's interface contract. It answers: what does this Function accept, what does it return, and what changed across versions?

Mandatory sections (see template):
1. **Identity** — name, version, one-sentence description
2. **Inputs table** — every input field with type, required/optional, default, and what the default does
3. **Outputs table** — every output field with type, success value, and failure value
4. **Behavior notes** — edge cases that aren't obvious from the field types (e.g., "if `depth` is `basic`, `canonical_url` is always `null`")
5. **Change log** — one row per version with: version number, date, type (additive/breaking), and what changed

Don't put usage examples in the spec. That goes in the usage doc.

---

## The usage doc — what it contains and why

The usage doc is what you hand to someone building a Clay table. It answers: how do I reference this Function, what does the output look like in a cell, and when should I use the `full` depth vs `basic`?

Mandatory sections (see template):
1. **One-liner** — what this Function does in plain language
2. **When to use it** — 2–3 concrete use cases
3. **Input reference** — a quick table (field, what to pass, example value)
4. **Output reference** — what the output object looks like in a Clay cell with a real example value
5. **Common patterns** — 2–3 concrete formulas or downstream patterns that consume this Function's output

Don't repeat the spec's full type/behavior details in the usage doc. Link to the spec for that.

---

## Step-by-step

1. Copy `assets/templates/function-spec.md` to `functions/<function_name>/spec.md`.
2. Fill every section. If a section doesn't apply, delete it — don't leave blank placeholders.
3. Copy `assets/templates/usage-doc.md` to `functions/<function_name>/usage.md`.
4. Fill it after your first real run with real output values.
5. Create `functions/<function_name>/backlog.md`. Seed it from canvas Q10 — anything deferred during design goes here. Format: a plain list with a one-line reason for each item. An empty backlog is fine; the file still ships so future maintainers have a place to add items.
6. Check: does the usage doc link to the spec? Does the spec have at least one entry in the change log? Does the backlog file exist?
7. **Haiku QA pass.** Spin up a Haiku agent with the following prompt, substituting the actual function name:

   > You are QA-reviewing the documentation for the Clay Function `<function_name>`. Read all four files: `functions/<function_name>/canvas.md`, `functions/<function_name>/spec.md`, `functions/<function_name>/usage.md`, and `functions/<function_name>/backlog.md`. Check for: (1) all required fields filled — no blank placeholders left, (2) Clay UI names used consistently across canvas and spec (Title Case, no snake_case field names), (3) Clay descriptions present for every input, (4) Exclusions section either filled or deleted, (5) Clay types are valid (Text, Number, Boolean, URL, Date — no JSON), (6) every output field has a failure value, (7) usage doc has at least one real output example (not a placeholder), (8) backlog.md exists. Return a numbered list of findings with file and section for each. Severity: blocker / moderate / minor.

   Share the Haiku findings with the user before moving to `sop/04-pre-publish-checklist.md`. Fix any blockers before proceeding.

---

## What "testable steps" means for docs

Every step in the usage doc that says "do X" must be something the reader can verify. Examples:

- "Pass the raw domain string from your CRM column" — testable. They can look at the column and see if it's a raw domain.
- "Make sure the URL is valid" — not testable. Valid how? What does invalid look like?

If you find yourself writing "make sure" or "ensure that," rewrite it as a concrete check or a concrete action.

---

## ADRs — when to write one

Log an ADR in the spec's **ADRs section** (not a separate file) when you made a non-obvious design call a future maintainer might question. Examples that warrant one:

- You chose not to include a reasoning string after initially planning to
- You chose one Function over two primitives, or vice versa
- You chose an enum status field over a boolean because you anticipated more than two states
- You changed the output shape in a way that required a version bump

Examples that don't warrant an ADR:

- Naming a field
- Choosing a default value that's obvious from the behavior
- Adding a new field (note it in the change log only)
