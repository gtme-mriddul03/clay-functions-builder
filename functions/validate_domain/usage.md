# Usage — `validate_domain_v1`

For Clay table builders. Fill output examples after first real run.

---

## What this does

Normalizes a raw domain string and determines whether it resolves to an active business website. Returns a validity boolean, a clean bare domain, and the working URL for downstream agents.

---

## When to use it

- As the first step in any web-scraping workflow — gates all downstream Claygent and LLM enrichment
- When normalizing messy domain inputs that may include protocols, paths, www prefixes, or ports
- When filtering out parked, placeholder, or unreachable domains before running expensive enrichment

---

## Clay column setup

**Add this function's column as:** `Domain Validator`

**Before this column:** add a Clay formula column that detects raw IP inputs (e.g., input matches `^\d{1,3}(\.\d{1,3}){3}$`) and outputs `is_valid: false` for those rows. The Claygent skips rows where the formula fires.

---

## Inputs

| Field | What to pass | Example |
|---|---|---|
| `domain` | Raw domain from your table — any format | `https://stripe.com`, `stripe.com`, `www.stripe.com/pricing` |
| `excluded_tlds` | List of TLDs to auto-fail, or leave empty | `[".gov", ".edu"]` |

`excluded_tlds` is optional. Leave it null if you have no TLD restrictions — all TLDs are checked by default.

---

## Output examples

**Valid domain (resolves on https://www.):**
```json
{
  "is_valid": true,
  "isolated_domain": "stripe.com",
  "working_domain": "https://www.stripe.com"
}
```

**Valid domain (resolves on http:// only):**
```json
{
  "is_valid": true,
  "isolated_domain": "oldsite.com",
  "working_domain": "http://oldsite.com"
}
```

**Invalid — parked or for-sale domain:**
```json
{
  "is_valid": false,
  "isolated_domain": "oldcompany.com",
  "working_domain": null
}
```

**Invalid — excluded TLD:**
```json
{
  "is_valid": false,
  "isolated_domain": "example.gov",
  "working_domain": null
}
```

**Invalid — unreachable across all variants:**
```json
{
  "is_valid": false,
  "isolated_domain": "deadco.com",
  "working_domain": null
}
```

---

## Common patterns

**Gate downstream scraping — only run if valid:**
```
Is Valid Domain? = true
```

**Pass working domain to all downstream agents:**
Use `Working Domain` as the URL input to every downstream Claygent or scraping step — never pass the raw input domain.

**Filter before routing to outreach:**
```
IF(Is Valid Domain? = true, route to outbound, skip)
```

**Use isolated domain for deduplication:**
`Isolated Domain` is populated even when `is_valid` is false — use it to deduplicate rows before this function runs, or to log which domain was attempted.

---

## What counts as invalid

Hard disqualifiers baked in — not configurable:
- Parked or for-sale domains (GoDaddy, Sedo, Afternic, Namecheap holding pages)
- Under construction, coming soon, future home of pages
- cPanel / Plesk / hosting provider default landing pages
- Raw IP address inputs (caught by upstream formula before Claygent runs)

**Not a disqualifier:** maintenance mode, temporarily unavailable, login-only pages — these are real business sites.

---

## Full interface

See [functions/validate_domain/spec.md](spec.md).
