# Function Spec — `validate_domain`

---

## Identity

| Field | Value |
|---|---|
| Name | `validate_domain` |
| Current version | v1 |
| Clay UI name | `validate_domain_v1` |
| One-line description | Resolves a raw domain string to the working URL variant that serves real business content. |
| Status | `active` |

---

## Inputs

| Field | Type | Required | Default | Default behavior |
|---|---|---|---|---|
| `Company Domain` | string | yes | — | Raw input — any format (with/without protocol, www, path, port) |
| `TLDs Exclusions` | string | no | null | Comma-separated TLDs that auto-fail with `is_valid: false` before Claygent runs; e.g. `.gov,.edu` |

---

## Outputs

| Field | Type | Success value | Failure value | Clay column name | Clay type |
|---|---|---|---|---|---|
| `is_valid` | boolean | `true` | `false` | `Is Domain Valid?` | Boolean |
| `isolated_domain` | string | `stripe.com` | best-effort parse; `null` if unparseable | `Isolated Domain` | Text |
| `working_domain` | string | `https://www.stripe.com` | `null` | `Working Domain` | Text |

`isolated_domain` is populated even when `is_valid` is false — callers may need the clean domain for deduplication or logging regardless of validity.

---

## Clay column names

| Output field | Clay column | Convention applied |
|---|---|---|
| `is_valid` | `Is Domain Valid?` | Boolean outputs → `Is [X]?` |
| `isolated_domain` | `Isolated Domain` | Title Case |
| `working_domain` | `Working Domain` | Title Case |

Internal agent columns:

| Agent | Clay column |
|---|---|
| HTTPS primary | `Claygent: Validate HTTPS Domain` |
| HTTPS fallback | `Fallback Claygent: Validate HTTPS Domain` |
| HTTP primary | `Claygent: Validate HTTP Domain` |
| HTTP fallback | `Fallback Claygent: Validate HTTP Domain` |
| HTTPS://WWW primary | `Claygent: Validate HTTPS://WWW Domain` |
| HTTPS://WWW fallback | `Fallback Claygent: Validate HTTPS://WWW Domain` |
| HTTP://WWW primary | `Claygent: Validate HTTP://WWW Domain` |
| HTTP://WWW fallback | `Fallback Claygent: Validate HTTP://WWW Domain` |

The column calling this Function: **`Domain Validator`**

---

## Agent architecture

Model: `gpt-4.1-nano` (Claygent Nano)

The function uses 8 Claygent columns — one primary + one fallback per URL variant — gated by Clay's conditional run logic. No single multi-visit prompt.

**URL variant sequence (runs in priority order, stops at first valid):**

| Step | Primary column | Fallback column | Boolean gate |
|---|---|---|---|
| 1 | `Claygent: Validate HTTPS Domain` | `Fallback Claygent: Validate HTTPS Domain` | `Is HTTPS Domain Valid?` |
| 2 | `Claygent: Validate HTTP Domain` | `Fallback Claygent: Validate HTTP Domain` | `Is HTTP Domain Valid?` |
| 3 | `Claygent: Validate HTTPS://WWW Domain` | `Fallback Claygent: Validate HTTPS://WWW Domain` | `Is HTTPS://WWW Domain Valid?` |
| 4 | `Claygent: Validate HTTP://WWW Domain` | `Fallback Claygent: Validate HTTP://WWW Domain` | `Is HTTP://WWW Domain Valid?` |

**Conditional run gates:**
- Each primary Claygent runs only if the previous variant's boolean gate is false AND `Isolated Domain` is populated.
- Each fallback Claygent runs only if its corresponding primary's `stepsTaken.length > 1` (Nano violated the one-visit rule).
- Each boolean gate formula: `primary?.response || fallback?.response`

**Final consolidation (formula columns):**
- `Is Domain Valid?` — `true` if any of the 4 boolean gates is true
- `Working Domain` — formula reconstructs the URL from `Isolated Domain` + protocol prefix in priority order: `https://` → `http://` → `https://www.` → `http://www.` → empty string if all fail

**Domain isolation:**
- `Isolate Domain` — Clay native `normalize-url` action with `bareDomain` type. Runs only if `Is Domain Relevant? (TLD check)` is true.
- `Isolated Domain` — formula extracting `normalizedUrl` from the Isolate Domain action.

**Upstream formula gate (`Is Domain Relevant? (TLD check)`):**
Intercepts three cases before any Claygent runs:
1. **Raw IP inputs** — input matches IPv4/IPv6 pattern → false
2. **Missing TLD** — input has no dot → false
3. **Excluded TLDs** — domain's TLD is in `TLDs Exclusions` → false

---

## Behavior notes

**Input normalization:** strips protocol, www prefix, trailing paths, and ports → bare domain. Examples: `https://app.stripe.com/pricing` → `stripe.com`; `http://www.example.org:8080` → `example.org`.

**Structural disqualifiers (caught by upstream formula — Claygent never runs):** raw IP addresses (IPv4/IPv6), inputs with no TLD (no dot in input), and TLDs in `TLDs Exclusions`. All return `is_valid: false`, `working_domain: null`. `isolated_domain` is null for IPs and no-TLD inputs; populated with the stripped domain for excluded TLDs.

**URL variant order:** Four separate Claygent columns run sequentially, gated by Clay's conditional run logic. Each column makes exactly one visitWebpage call. The next variant only runs if the previous boolean gate is false. Priority: `https://` → `http://` → `https://www.` → `http://www.`.

**Validity judgment — positive evidence framing:** Each Claygent column asks one question: "does this content show genuine evidence of a real operating business?" Genuine evidence includes: company name, services offered, navigation menu, contact details, portfolio, team information, blog posts, or product descriptions. A page fails if it shows only: domain-for-sale or parked content, coming soon or under construction content, hosting provider default pages (cPanel/Plesk) with no real business content, or placeholder text with no real business information. This framing was chosen over a disqualifier blocklist because it handles novel invalid page types without requiring rule updates — finding one confirming signal is a simpler task for a constrained model than anticipating every possible invalid signal.

**200 OK with no extractable content:** treated as unreachable for that variant; next variant runs.

**Fallback columns:** Each URL variant has a fallback Claygent column that fires when `stepsTaken.length > 1` — meaning Nano made more than one visitWebpage call and violated the one-visit rule. The fallback re-runs the same prompt on the same URL. The boolean gate for each variant is `primary?.response || fallback?.response`, so a valid result from either column counts.

**Working Domain derivation:** `Working Domain` is constructed by a formula column reading which boolean gate is true first, then prepending the corresponding protocol prefix to `Isolated Domain`. The model never self-reports the URL — this eliminates hallucination where models reconstructed a plausible URL from the domain input rather than from what they actually visited.

**Redirects:** followed. `Working Domain` = the protocol+domain that triggered the redirect (formula-derived from `Isolated Domain`), not the redirect-resolved canonical URL. See backlog for redirect-resolution improvement.

**Maintenance mode / temporarily unavailable:** NOT a disqualifier. These are real business sites temporarily down. Claygent returns Valid.

**Login-only pages:** Valid. A domain showing only a login form is a real business site. See backlog for future `requires_auth` flag consideration.

**Subdomain stripping:** all subdomains including non-www are stripped to the root domain (`app.stripe.com` → `stripe.com`).

---

## Composition

Standalone. No sub-functions called.

---

## Version increment triggers

A v2 is cut when:
- Any output field (`is_valid`, `isolated_domain`, `working_domain`) is removed or renamed
- `is_valid` type changes from boolean
- `Company Domain` required input is removed or renamed
- Validity judgment criteria change in a way that flips existing valid → invalid decisions for current callers
- `isolated_domain` stripping logic changes (e.g., subdomains start being preserved)

Adding new optional output fields or optional inputs with behavior-preserving defaults → additive, no version bump.

---

## Change log

| Version | Date | Type | What changed |
|---|---|---|---|
| v1 | 2026-04-28 | initial | Initial publish |
| v1.1 | 2026-04-29 | additive | Architecture upgraded: Nano model, column-per-variant with fallbacks, positive evidence framing, formula-derived Working Domain |

---

## ADRs

**`validate_domain` chosen over `check_domain`**
`check` is too generic — with `domain` as the noun it tells callers nothing about what kind of check. `validate` signals a structured validity judgment: resolving URL variants, applying hard disqualifiers, and returning a usable working URL. Matches the SOP 01 canonical naming example.

**Boolean chosen over enum for `is_valid`**
Downstream agents gate on proceed/don't-proceed only — they don't branch on failure reason (parked vs. unreachable vs. placeholder). A status enum would go unused in v1. If a future caller needs to distinguish failure reasons, add a `failure_reason` enum as an additive output field then.

**`working_domain` outputs the attempted URL, not the redirect-resolved canonical URL**
Implementing redirect resolution in v1 adds Claygent prompt complexity without a confirmed caller complaint. Callers re-following a redirect is a minor inefficiency, not a failure. Logged in backlog for v2 consideration.

**Raw IP detection via upstream Clay formula, not Claygent**
A regex formula is cheaper and deterministic for structural pattern matching. Claygent is reserved for content judgment. Sending an IP address to Claygent wastes a run on a case that can be caught for free upstream.

**`excluded_tlds` at TLD granularity only — full-domain exclusions declined**
Caller confirmed TLD-level exclusion covers all known use cases. Full-domain exclusions would require a different input type and add interface complexity with no current need. If a future caller needs to exclude specific domains, add `excluded_domains: string[], optional, null` as an additive input.

**`isolated_domain` strips all subdomains including non-www**
Caller confirmed: strip to bare root domain (`app.stripe.com` → `stripe.com`). Preserving non-www subdomains added to backlog. If a future caller needs subdomain-aware normalization, this becomes a breaking change requiring v2.

**`gpt-4.1-nano` (Claygent Nano) chosen as model**
The task is decomposed to its atomic unit — one URL visit, one binary evaluation, one output. At this level of simplicity Nano performs reliably and the cost advantage at scale is decisive. Higher-tier models were not justified for a task this constrained.

**Column-per-URL-variant architecture over single multi-visit prompt**
Nano cannot maintain an accurate running log across sequential tool calls. Rather than fight that ceiling with prompt complexity, sequencing is offloaded to Clay's conditional run logic, which is deterministic and free. Each column knows only one URL and one job. Cost benefit: each prompt is short (one URL, one visit, one evaluation) → lower token cost per call. More importantly, if the domain validates on the first or second variant, all remaining columns are skipped by their conditional run gates — zero cost for those columns. A domain that resolves on `https://` costs one Claygent credit; one that fails all four costs eight.

**Positive evidence framing over disqualifier blocklist**
Each Claygent prompt asks "does this show genuine evidence of a real operating business?" rather than maintaining a list of invalid signals. Positive framing handles novel invalid page types without requiring rule updates — finding one confirming signal is simpler and more robust for a constrained model than anticipating every possible invalid signal.

**Fallback Claygent column per variant**
Each URL variant has a fallback column that fires when `stepsTaken.length > 1`, indicating Nano violated the one-visit rule. This is a self-healing quality control mechanism that costs a credit only when the primary column misbehaves, which is infrequent.

**Working Domain derived by formula, not model self-report**
The model never outputs the URL it visited. A formula column reconstructs the working URL from `Isolated Domain` + the protocol prefix of whichever boolean gate fired first. This eliminates the single largest source of hallucination observed in earlier iterations, where models consistently reconstructed a plausible URL from the domain input rather than from what they actually visited.

**HTTPS-first variant priority order**
Organized-sector companies — startups, funded companies, companies with meaningful employee counts — predominantly serve on HTTPS. Smaller or older companies more commonly serve on HTTP only. Since the primary targets for outbound are organized-sector companies, `https://` is tried first: it produces a valid result earlier in the sequence for the majority of targets, which maximizes early exits and minimizes credits spent. The HTTP variants (steps 2 and 4) serve as fallbacks for smaller companies that haven't migrated to HTTPS.

**Clay native `normalize-url` used for domain isolation**
Domain isolation (stripping protocol, www, path) is handled by Clay's built-in `normalize-url` action with `bareDomain` type, not by Claygent. A native action is cheaper, deterministic, and not subject to model variability.

---

## Known consumers

- All scraping tables (10+ active)

---

## Related

- Canvas: [functions/validate_domain/canvas.md](canvas.md)
- Usage doc: [functions/validate_domain/usage.md](usage.md)
- Backlog: [functions/validate_domain/backlog.md](backlog.md)
