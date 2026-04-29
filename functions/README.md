# Functions

Register of shipped Clay Functions. Add a row here when a function goes live.

Specs and usage docs live in [`../skill/docs/`](../skill/docs/). Canvas drafts live in [`../skill/working/`](../skill/working/).

## Shipped functions

| Function | Version | Spec | Usage doc | Status |
|---|---|---|---|---|
| `check_public_listing` | v1 | [spec.md](check_public_listing/spec.md) | [usage.md](check_public_listing/usage.md) | active |
| `validate_domain` | v1 | [spec.md](validate_domain/spec.md) | [usage.md](validate_domain/usage.md) | active |

---

## Your first session: how to build a function

### 1. Invoke the skill
Start any new session with:
```
/clay-functions-v2
```
This loads the routing table, opinionated positions, and guardrails. Don't skip this — the skill's gates (reasoning strings, confidence scores, depth toggles) are where the framework earns its keep.

### 2. Run SOP 00 — should you build this at all?
Open [`../skill/sop/00-when-to-build.md`](../skill/sop/00-when-to-build.md). Four gates. All four must pass. The gate that bites most often: **can you name a second concrete caller today?** "Probably will need it" doesn't count.

If you pass all four, continue. If not, stop — you're not building a Function yet.

### 3. Fill the canvas
Copy the canvas template and fill it before writing a single line of Clay workflow:
```
skill/assets/templates/function-canvas.md → skill/working/<function-name>-canvas.md
```
The canvas has six questions. The two that matter most: what's the exact one-sentence description (verb, input→output, no "and"), and what does the output look like when the check *fails*? If you can't answer both, the interface isn't clear yet.

### 4. Design the interface (SOP 01)
[`../skill/sop/01-design.md`](../skill/sop/01-design.md) locks the contract. Key calls:
- Use a status enum (not a boolean) if downstream consumers need to know *why* something failed
- Include a reasoning string only if a downstream workflow step *branches on* its content
- No confidence scores unless a named consumer has a threshold gate that reads the value
- Extract a primitive only if a *second Function* (not a second table) needs the sub-step

### 5. Write the spec
```
skill/assets/templates/function-spec.md → skill/docs/<function-name>-spec.md
```
Fill it before you build the Clay workflow. If you can't fill the outputs table, you don't know what you're building.

### 6. Build in Clay
Build the Function in Clay. UI name must be `<function-name>_v1`. Internal implementation is up to you — the framework governs the interface, not the workflow body.

### 7. Run the pre-publish checklist
[`../skill/sop/04-pre-publish-checklist.md`](../skill/sop/04-pre-publish-checklist.md) before hitting publish. Every gate must pass. The ones most likely to catch something: load-bearing field check, semantic-drift lock, and the spec completeness gate.

### 8. Publish, then fill the usage doc
```
skill/assets/templates/usage-doc.md → skill/docs/<function-name>-usage.md
```
Fill the usage doc *after* your first real run with real output values — not before.

### 9. Update this file
Add a row to the Shipped functions table above with links to the spec and usage doc.

---

## File locations at a glance

| Artifact | Path |
|---|---|
| Canvas draft | `skill/working/<function-name>-canvas.md` |
| Function spec | `skill/docs/<function-name>-spec.md` |
| Usage doc | `skill/docs/<function-name>-usage.md` |
| ADR (if needed) | `skill/decisions/<YYYYMMDD>-<function-name>-<slug>.md` |
