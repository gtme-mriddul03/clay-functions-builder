# SOP 01 — Designing a Function's Scope and Interface

Do this after passing `sop/00-when-to-build.md` and filling the canvas. This SOP takes your draft canvas and locks the interface.

---

## Step 1 — Confirm scope with the one-sentence test

Write the Function name and a one-sentence description of what it does. The sentence must:

- Start with a verb
- Describe the input → output transformation
- Contain no "and," no "or," no "also"

If you can't write that sentence, the scope isn't clear yet. Go back to the canvas.

**Pass:** `validate_domain: Takes a raw URL string and returns whether it resolves to a live, non-parked domain.`

**Fail:** `validate_domain: Takes a URL and checks if it's valid and resolves and whether it's parked.` (three concerns)

---

## Step 2 — Design inputs

For each input field, answer:

1. What type is it? (string, boolean, number, enum — pick one, not "string or number")
2. Is it required or optional?
3. If optional, what's the default value and what behavior does the default produce?

**Rules:**
- Mark an input required only if the Function cannot produce any output without it.
- If an input affects behavior depth or behavior variant (e.g., "check HTTPS only" vs "check HTTPS + parked"), make it an optional enum with a documented default. Don't make callers pass it every time.
- No more than 5 inputs. If you need more, the scope is probably wrong.
- Don't use boolean flags to toggle between fundamentally different behaviors. Use an enum.

**Example input table:**

| Field | Type | Required | Default | Effect of default |
|---|---|---|---|---|
| `url` | string | yes | — | — |
| `depth` | enum: `basic`, `full` | no | `basic` | HTTP check only; skip parked detection |

---

## Step 3 — Design outputs

1. List every output field you intend to return.
2. For each field: name, type, and what value it takes when the check fails vs passes.
3. Answer the reasoning string question: will any downstream consumer **branch on** the reasoning text? If yes, include it. If no, omit it.
   - **"Branches on" defined:** the next workflow step (a Claygent prompt, a formula, another Function, a CRM sync filter) reads the value of the reasoning string and behaves differently based on its content. "A reviewer might want to read it" is not branching. "It might be useful someday" is not branching. The reader has to be a downstream *workflow step*, and the read has to change behavior.
4. Answer the confidence score question: is there a downstream threshold gate that reads this score? If yes, include it. If no, omit it.

**Rules:**
- Keep outputs flat. One level of nesting is OK for a coherent sub-object with 3+ fields. No deeper.
- Every output field must have a defined failure value (not just a success value). `null` is a valid failure value; document it explicitly.
- Boolean status fields are fine for simple pass/fail. If the downstream consumer needs to distinguish *why* it failed, use an enum status field instead of a boolean.
- Name fields for what they contain, not how they were computed. `is_live` not `http_check_result`.

**Example output table:**

| Field | Type | Success value | Failure value |
|---|---|---|---|
| `is_valid` | boolean | `true` | `false` |
| `status` | enum: `live`, `parked`, `unreachable`, `redirect_mismatch` | `live` | one of the others |
| `canonical_url` | string | resolved URL | `null` |
| `reasoning` | string | explanation | explanation of failure |

---

## Step 4 — Composition check

Look at your input and output design. Ask:

- Is any part of this Function's logic independently useful to a different Function I already have or am planning?
- If I extracted that part as a primitive, would the primitive appear in 2+ Functions?

If yes to both: extract the primitive, run SOP 00–01 on it, and reference it from this Function.

If no: all the logic stays inside this Function.

Do not extract a primitive "just in case." Extract it only when the second use is concrete and in-hand.

---

## Step 5 — Lock the interface

Write the final interface block. This is the contract. Once you publish v1, this is what you cannot change without a version bump.

```
Function: <name>
Version: 1
Inputs:
  - <field>: <type>, required/optional, default: <value>
Outputs:
  - <field>: <type>
  - ...
Breaking change policy: removing or renaming any field, changing any field's type, or changing input defaults that affect existing callers.
Additive change policy: adding new optional output fields.
```

Once this block is written and you're building, treat it as locked. Any change to it before publish is fine. Any change to it after publish that would break consumers requires a new version.

---

## Step 6 — Name it

Function names:

- Lowercase, underscores, no hyphens
- Verb_noun format: `validate_domain`, `normalize_url`, `check_email`
- Version suffix in the Clay UI name: `validate_domain_v1` (not in the internal function name in specs — the spec carries the version field separately)

If your name has more than two words, the scope might be too broad. Check it.
