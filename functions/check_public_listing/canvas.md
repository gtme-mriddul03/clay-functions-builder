# Function Canvas — `check_public_listing`

---

## Q1 — What does this Function do? (one sentence, verb-first, no "and")

**Answer:** Takes a company's name and domain and returns whether the company is listed on a major public stock exchange.

---

## Q2 — Who uses this?

List every Clay table or workflow that will reference this Function. If you can only name one and no second is scheduled, stop — build it inline instead.

**Current consumers:** Raw Leads List tables (one per client/industry run)

**Upcoming consumers:** Additional client Raw Leads List tables across industries

---

## Q3 — What goes in?

| Field | Type | Required | Default | Default behavior |
|---|---|---|---|---|
| `company_name` | string | yes | — | — |
| `domain` | string | yes | — | — |
| `linkedin_url` | string | no | null | Skipped; used only to disambiguate when name/domain are ambiguous |
| `location` | string | no | null | Claygent checks all major exchanges; with location, scopes to regional exchanges |

**Exclusion list:** SPACs, shell companies, blank check companies, ADRs, ETFs/funds, BDCs, liquidating trusts, REITs — all baked in. Not configurable. Always return `false`.

**Pass-throughs:** The function fetches exchange listing data via Claygent. Ticker is the natural pass-through but callers never have it at this stage of the workflow — not added.

---

## Q4 — Who consumes the output and what do they need?

**Consumer:** Raw Leads List table

**Fields needed:** `is_publicly_traded` (used to filter or score rows)

---

## Q5 — What comes out?

| Field | Type | Success value | Failure value | Clay column name |
|---|---|---|---|---|
| `is_publicly_traded` | boolean | `true` | `false` | `Is Publicly Listed?` |

**Reasoning string needed?** No — downstream use is filtering only. Claygent's run log covers audit needs.

**Confidence score needed?** No — no threshold gate downstream.

---

## Q6 — Agent architecture

**Inferred design:** Single Claygent column that always checks all major exchanges.

| Agent | Internet | Runs when | Skipped when |
|---|---|---|---|
| `Claygent: Check Listing` | yes | always | never |

Always checks all major exchanges (NYSE, NASDAQ, LSE, TSX, ASX, NSE, BSE, SGX) regardless of location. Incorporation country is not a reliable proxy for listing exchange — a company can be incorporated in one country but listed on a completely different exchange (e.g., an Indian company listed on NASDAQ).

---

## Q7 — Is anything here independently useful to a different Function?

**Answer:** No. The exchange lookup logic is specific to this function's purpose. No extraction warranted.

---

## Q8 — Primary failure case

**Failure case:** Company name and domain don't match any known listed entity (company is private, acquired, or data is too thin to resolve).

**Output in that case:** `is_publicly_traded: false`

---

## Q9 — Clay column names

**Function column:** `Is Publicly Listed?`

**Internal agent columns:**
- `Claygent: Check Listing`

---

## Go / No-Go

- [x] Q1 has no "and"
- [x] Q2 has ≥2 named consumers
- [x] Q3 exclusion list and pass-throughs answered
- [x] Q4 consumer named before outputs designed
- [x] Q5 has failure values for every field and Clay column names filled
- [x] Q6 answered if agents are involved
- [x] Q7 answered
- [x] Q8 answered
- [x] Q9 follows naming-conventions.md
