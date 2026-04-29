# SOP 01 — Designing a Function's Scope and Interface

Do this after passing `sop/00-when-to-build.md` and filling the canvas.

**Batching rule:** send all questions within a step together — see SKILL.md. No single-question round-trips.

---

## Step 1 — Confirm scope with the one-sentence test

Write the Function name and a one-sentence description. The sentence must:

- Start with a verb
- Describe the input → output transformation
- Contain no "and," "or," or "also"

If you can't write that sentence, scope isn't clear. Go back to the canvas.

**Pass:** `validate_domain: Takes a raw URL string and returns whether it resolves to a live, non-parked domain.`

**Fail:** `validate_domain: Takes a URL and checks if it's valid and resolves and whether it's parked.` (three concerns)

---

## Step 2 — Design inputs

For each input field, answer:

1. What type? (string, boolean, number, enum — pick one)
2. Required or optional?
3. If optional: what's the default, and what does it do?
4. Clay description: a short phrase (≤ 15 words) to paste directly into Clay's input description field — this is what anyone adding the Function to a table sees.

**Rules:**
- Use Clay UI names (Title Case) for every field — these are the exact names as they appear in the Function UI.
- Required only if the Function cannot produce any output without it.
- Behavior variants → optional enum with documented default. Not a boolean flag.
- No more than 5 inputs. If you need more, check scope first — but defend the extra if justified.
- Don't use boolean flags to toggle between fundamentally different behaviors. Use an enum.

**Two standing questions for every Function:**

1. **Exclusion list:** Are there categories that always disqualify, regardless of the positive criteria? If yes, add an exclusion input as optional (default: null). Name it in Title Case matching what it excludes (e.g., `TLDs Exclusions`, `Industry Exclusions`). Exclusions go in the Exclusions section of the spec — not mixed into the main Inputs table.
2. **Pass-throughs:** What data does this Function fetch internally? For each fetch, ask: could a caller already have this? If yes, add it as an optional input so the fetch is skipped. This is the most common source of missed inputs.

**Example:**

| Clay UI Name | Type | Required | Default | Default behavior | Clay description |
|---|---|---|---|---|---|
| Company Domain | string | yes | — | — | Raw domain or full URL from your CRM |
| Check Depth | enum: `basic`, `full` | no | `basic` | HTTP check only; skip parked detection | Set to `full` to also detect parked domains |

---

## Step 3 — Downstream consumer

Before designing outputs, answer: **what table, formula, or downstream Function consumes this output, and what fields does it need?**

Design outputs from the consumer backward — only include fields a named consumer actually reads. Fields with no named consumer are omitted.

---

## Step 4 — Design outputs

1. List every output field a named consumer needs.
2. For each: name, type, success value, failure value.
3. Reasoning string: will any downstream step branch on its content? If yes, include. If no, omit.
4. Confidence score: is there a threshold gate downstream that reads it? If yes, include. If no, omit.

**Rules:**
- Flat by default. One level of nesting OK for a coherent sub-object with 3+ related fields.
- Every field needs a defined failure value. `null` is valid; document it.
- Boolean for simple pass/fail. Enum if the consumer needs to distinguish *why* it failed.
- Name fields for what they contain, not how they were computed.

**Example:**

| Clay UI Name | Type | Success value | Failure value |
|---|---|---|---|
| Is Valid | boolean | `true` | `false` |
| Status | enum: `live`, `parked`, `unreachable` | `live` | one of the others |
| Canonical URL | string | resolved URL | `null` |

---

## Step 5 — Agent and formula architecture

If this Function has no agents and no load-bearing formula columns, skip to Step 6 — this step does not apply.

If this Function uses LLM agents or load-bearing formula columns, infer and document the full architecture. Do not ask the user to define this — derive it from the input/output design and present it for confirmation.

**What counts as load-bearing:** conditional gates, consolidation formulas, URL reconstruction, and any formula column whose output is read by another row or consumed as an output. These are first-class architecture components — document them alongside agent rows.

Infer per SKILL.md agent architecture rules: internet access from purpose, parallelism from dependencies, pass-throughs from Step 2 inputs. For formula columns, note what they gate or consolidate. Present the inferred design as a one-sentence summary and ask the user to confirm or correct.

**Validity pattern — choose one before writing agent prompts:**

Two patterns exist for content-judgment agents (e.g., "is this a real company?"). Pick the right one at design time:

- **Disqualifier blocklist:** agent looks for signals that disqualify a record. Use when the invalid set is small, well-defined, and unlikely to grow (e.g., "reject if domain is a free email provider").
- **Positive evidence framing:** agent looks for one confirming signal. Use when the valid set is easier to characterize than the invalid set, or when the model tends to hallucinate disqualifiers on noisy data (e.g., "confirm if there is a LinkedIn company page matching this name and location").

**Default to positive evidence framing** unless you can enumerate the full invalid set at design time. If you start writing a blocklist and can't define its edges, switch — that uncertainty means the valid set is easier to characterize.

---

## Step 6 — Composition check

- Is any part of this Function's logic independently useful to a *different* Function (not just another table)?
- If extracted as a primitive, would it appear in 2+ Functions?

Yes to both: extract the primitive, run SOP 00–01 on it, reference it from this Function.
No: all logic stays inside this Function.

Do not extract primitives speculatively.

---

## Step 7 — Lock the interface

```
Function: <name>
Version: 1
Inputs:
  - <field>: <type>, required/optional, default: <value>
Outputs:
  - <field>: <type>
Version increment triggers: <what would force a v2>
Breaking change policy: removing/renaming any field, changing any field's type, changing input defaults that affect existing callers.
Additive change policy: adding new optional output fields or optional inputs with behavior-preserving defaults.
```

Once written and published, any change that breaks a consumer requires a new version.

---

## Step 8 — Name it

**Function name (folder / code reference):**
- Lowercase, underscores: `check_icp_industry`
- Verb_noun format. Two words max — more usually means scope is too broad.
- See `references/naming-conventions.md` for verb guidance — avoid weak verbs like `check`, `get`, `run`.

**Clay UI name (what the Function is published as in Clay):**
- Version suffix: `check_icp_industry_v1`

**Function folder:**
- `functions/check_icp_industry/`

**Field names (inputs and outputs):**
- Use Clay UI names (Title Case) everywhere — in the spec, canvas, and when building in Clay. There is no separate internal field name. What you write in the spec IS what you type into Clay.

**Clay column names (for columns that call or reference this Function in parent tables):**
- Follow `references/naming-conventions.md`
- The column calling the Function → Title Case name matching its purpose
- Boolean outputs → `Is [X]?`
- Agent columns inside the Function → `Claygent: [Verb] [Subject]` or `LLM: [Verb] [Subject]`
- Formula columns inside the Function → plain descriptive Title Case (e.g., `Consolidate Scores`, `Reconstruct URL`)

When naming is done, copy `assets/templates/function-spec.md` to `functions/<verb_noun>/spec.md` and fill all sections.
