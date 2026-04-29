# Function Spec — `check_public_listing`

---

## Identity

| Field | Value |
|---|---|
| Name | `check_public_listing` |
| Current version | v1 |
| Clay UI name | `check_public_listing_v1` |
| One-line description | Returns whether a company is listed on a major public stock exchange. |
| Status | `active` |

---

## Inputs

| Field | Type | Required | Default | Default behavior |
|---|---|---|---|---|
| `company_name` | string | yes | — | — |
| `domain` | string | yes | — | — |
| `linkedin_url` | string | no | null | Skipped; used only to disambiguate when name/domain are ambiguous |
| `location` | string | no | null | Passed to Claygent as a disambiguation signal; all major exchanges are always checked regardless |

---

## Outputs

| Field | Type | Success value | Failure value | Clay column name | Clay type |
|---|---|---|---|---|---|
| `is_publicly_traded` | boolean | `true` | `false` | `Is Publicly Listed?` | Boolean |

---

## Clay column names

| Output field | Clay column | Convention applied |
|---|---|---|
| `is_publicly_traded` | `Is Publicly Listed?` | Boolean outputs → `Is [X]?` |

Internal agent columns:

| Agent | Clay column |
|---|---|
| Listing check | `Claygent: Check Listing` |

---

## Agent architecture

| Agent | Model | Internet | Runs when | Skipped when |
|---|---|---|---|---|
| `Claygent: Check Listing` | GPT-4.0 mini | yes | always | never |

Always checks all major exchanges (NYSE, NASDAQ, LSE, TSX, ASX, NSE, BSE, SGX) regardless of location. Location input is still used as a signal to assist disambiguation, not to scope exchanges.

---

## Behavior notes

**Baked-in exclusions (always `false`, not configurable):** SPACs, shell companies, blank check companies, ADRs, ETFs/closed-end funds, BDCs, liquidating trusts, REITs.

**Location as disambiguation signal, not scope:** `location` is passed to Claygent as a disambiguation signal, not to scope exchanges. A company can be incorporated in one country but listed on a different exchange entirely.

**Ambiguous location input:** `location` accepts free-form strings (country, state, or both). Claygent resolves the relevant exchanges. If location is too ambiguous to map, it falls back to global exchange check.

**Failure default:** When the company cannot be resolved to a known listed entity (private, acquired, insufficient data), returns `false`. There is no error state — the function always returns a boolean.

---

## Composition

Standalone. No sub-functions called.

---

## Version increment triggers

A v2 is cut when:
- `is_publicly_traded` is removed or renamed
- `is_publicly_traded` type changes from boolean
- Any required input (`company_name`, `domain`) is removed or renamed
- The baked-in exclusion list is modified in a way that would change results for existing callers (e.g., removing REITs from exclusions)

Adding a new optional output field (e.g., `exchange_name`, `reasoning`) or a new optional input with a behavior-preserving default → additive, no version bump.

---

## Change log

| Version | Date | Type | What changed |
|---|---|---|---|
| v1 | 2026-04-28 | initial | Initial publish |

---

## ADRs

**Reasoning string omitted**
The Raw Leads List caller uses `is_publicly_traded` for filtering only. Reasoning was requested for audit/human review, but Claygent's run log already surfaces this without adding it as an output field. Including it would add an unused column that drifts as prompts evolve. If a future table branches on reasoning content, add it as an additive output field then.

**Function named `check_public_listing`, not `is_publicly_listed`**
`is_*` naming self-documents boolean returns but breaks at scale: enrichment, scoring, and extraction functions can't follow it. Uniform `verb_noun` across all function types is more consistent than a pattern that only applies to booleans. The output column (`Is Publicly Listed?`) carries the boolean semantics — the function name carries the operation semantics. These are different layers.

**REITs baked in, not configurable**
REIT exclusion was raised as a candidate optional input during design. Baked in after confirming no known caller needs to include REITs. If a future caller does, adding `exclude_reits: boolean, optional, default: true` is a non-breaking additive change.

**Single agent regardless of location**
Initially designed as two agents (regional when location present, global when absent). Collapsed to one after recognising that incorporation country is not a reliable proxy for listing exchange — an Indian company can be listed on NASDAQ. Always checking all major exchanges avoids false negatives at negligible cost.

**GPT-4.0 mini chosen over GPT-5 Nano**
Compared both models. GPT-4.0 mini was cheaper and followed instructions more strictly in testing. GPT-5 Nano was not clearly better on accuracy for this task.

---

## Known consumers

- Raw Leads List (per client/industry run)

---

## Related

- Canvas: [functions/check_public_listing/canvas.md](canvas.md)
- Usage doc: [functions/check_public_listing/usage.md](usage.md)
