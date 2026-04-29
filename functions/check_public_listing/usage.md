# Usage — `check_public_listing_v1`

For Clay table builders. Output example should be filled after first real run.

---

## What this does

Checks whether a company is listed on a major public stock exchange and returns a boolean.

---

## When to use it

- Use when filtering a raw leads list to target only private companies
- Use when scoring ICP fit and public/private status is a disqualifier or qualifier
- Use when enriching an account list before routing to outbound sequences that need private-company targeting

---

## Clay column name

The column calling this Function should be named: **`Is Publicly Listed?`**

---

## Inputs

| Field | What to pass | Example |
|---|---|---|
| `company_name` | Company name as-is from your table | `Stripe` |
| `domain` | Root domain, no protocol | `stripe.com` |
| `linkedin_url` | LinkedIn company page URL if available | `https://www.linkedin.com/company/stripe` |
| `location` | Country, state, or both — free text | `United States` / `California, US` |

`linkedin_url` and `location` are optional. Pass them when available — they improve accuracy and scope the exchange lookup.

---

## Output example

**Success (listed):**
```json
{
  "is_publicly_traded": true
}
```

**Success (not listed or excluded):**
```json
{
  "is_publicly_traded": false
}
```

---

## Common patterns

**Filter to private companies only:**
```
Is Publicly Listed? = false
```

**Filter to public companies only:**
```
Is Publicly Listed? = true
```

**Use as ICP disqualifier in a scoring formula:**
```
IF(Is Publicly Listed? = true, 0, [base_score])
```

---

## Exclusions built in

The following entity types always return `false` regardless of listing status. Not configurable in v1:
SPACs, shell companies, blank check companies, ADRs, ETFs/closed-end funds, BDCs, liquidating trusts, REITs.

---

## Full interface

See [functions/check_public_listing/spec.md](spec.md).
