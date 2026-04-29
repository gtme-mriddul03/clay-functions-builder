# Function Canvas — `validate_domain`

---

## Q1 — What does this Function do? (one sentence, verb-first, no "and")

**Answer:** Resolves a raw domain string to the working URL variant that serves real business content.

---

## Q2 — Who uses this?

**Current consumers:** 10+ scraping tables across all active workflows — this is the universal first step before any web scraping.

**Upcoming consumers:** All future scraping functions.

---

## Q3 — What goes in?

| Field | Type | Required | Default | Default behavior |
|---|---|---|---|---|
| `Company Domain` | string | yes | — | Raw input — any format accepted (with/without protocol, www, path, port) |
| `TLDs Exclusions` | string | no | null | Comma-separated TLDs that auto-fail before Claygent runs; e.g. `.gov,.edu` |

**Exclusion list:** TLDs via `TLDs Exclusions` (dynamic, caller-controlled). Raw IP addresses (e.g., `192.168.1.1`) — baked in via upstream Clay formula, not a Claygent check.

**Pass-throughs:** The function fetches up to 4 URL variants internally. Callers never have the working URL variant before this runs — that's the point. No pass-throughs warranted.

---

## Q4 — Who consumes the output and what do they need?

**Consumer:** All downstream scraping agents and functions.

**Fields needed:** `is_valid` (gate: proceed or skip), `working_domain` (URL to visit), `isolated_domain` (clean domain for deduplication or display).

---

## Q5 — What comes out?

| Field | Type | Success value | Failure value | Clay column name |
|---|---|---|---|---|
| `is_valid` | boolean | `true` | `false` | `Is Domain Valid?` |
| `isolated_domain` | string | `stripe.com` | best-effort parse; `null` if unparseable | `Isolated Domain` |
| `working_domain` | string | `https://www.stripe.com` | `null` | `Working Domain` |

**Reasoning string needed?** No — downstream branches on `is_valid` (boolean gate) and uses `working_domain` directly. No step branches on text reasoning.

**Confidence score needed?** No — no threshold gate downstream.

---

## Q6 — Agent architecture

**Inferred design:** 8 Claygent Nano columns (one primary + one fallback per URL variant) gated by Clay's conditional run logic. Domain isolation via Clay native `normalize-url` action. Final consolidation by formula columns — no model self-reporting of URLs.

| Agent | Internet | Runs when | Skipped when |
|---|---|---|---|
| `Claygent: Validate HTTPS Domain` | yes | `Isolated Domain` is populated | upstream formula gate is false |
| `Fallback Claygent: Validate HTTPS Domain` | yes | primary `stepsTaken.length > 1` | primary behaved correctly |
| `Claygent: Validate HTTP Domain` | yes | `Is HTTPS Domain Valid?` is false | HTTPS variant already valid |
| `Fallback Claygent: Validate HTTP Domain` | yes | primary `stepsTaken.length > 1` | primary behaved correctly |
| `Claygent: Validate HTTPS://WWW Domain` | yes | `Is HTTP Domain Valid?` is false | earlier variant already valid |
| `Fallback Claygent: Validate HTTPS://WWW Domain` | yes | primary `stepsTaken.length > 1` | primary behaved correctly |
| `Claygent: Validate HTTP://WWW Domain` | yes | `Is HTTPS://WWW Domain Valid?` is false | earlier variant already valid |
| `Fallback Claygent: Validate HTTP://WWW Domain` | yes | primary `stepsTaken.length > 1` | primary behaved correctly |

---

## Q7 — Is anything here independently useful to a different Function?

**Answer:** No. URL resolution and validity judgment are tightly coupled — you can't validate without resolving, and you can't resolve without validating. No primitive extraction warranted.

---

## Q8 — Primary failure case

**Failure case:** All 4 URL variants are unreachable or return hard-disqualified content (parked, placeholder, or under construction).

**Output in that case:** `is_valid: false`, `working_domain: null`, `isolated_domain: best-effort parse or null`

---

## Q9 — Clay column names

**Function column:** `Domain Validator`

**Key output columns:**
- `Is Valid Domain?`
- `Isolated Domain`
- `Working Domain`

**Internal agent columns:**
- `Claygent: Validate Domain`

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
