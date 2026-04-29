# Backlog — `validate_domain`

Improvements deferred from v1. Evaluate for v2 when a concrete caller need arises. Do not implement speculatively.

---

## 1. Redirect-resolved `working_domain`

Currently `working_domain` outputs the URL variant that was attempted (e.g., `http://oldcompany.com`), not the final destination after redirect (e.g., `https://newcompany.com`). Downstream scraping agents that receive the attempted URL will re-follow the redirect themselves.

**Why deferred:** Redirect resolution adds Claygent prompt complexity without a confirmed caller complaint. Re-following a redirect is a minor inefficiency, not a failure.

**Trigger to implement:** A downstream agent produces degraded output because it re-followed a redirect to an unexpected destination. Or a caller needs the canonical URL for deduplication (e.g., `oldcompany.com` and `newcompany.com` resolve to the same site and need to be collapsed).

**Proposed change:** Output the final resolved URL after all redirects as `working_domain`. This is a breaking change if `working_domain` currently returns the attempted URL in a way callers depend on — version to v2. Additive if shipped as a new field `canonical_url` alongside `working_domain`.

---

## 2. Explicit maintenance mode non-disqualifier documentation

"Site in maintenance mode" and "temporarily unavailable" are currently treated as Valid — not hard disqualifiers. This is the right call. But the Claygent prompt does not explicitly document it, which means a future prompt revision could accidentally flip these to Invalid.

**Why deferred:** Low risk in v1 since Claygent uses judgment and the intent is clear in the reference examples. Explicit documentation adds prompt length for a low-frequency edge case.

**Trigger to implement:** A maintenance-mode domain starts getting flagged as Invalid in production. Or the Claygent prompt is significantly revised and this edge case needs re-anchoring.

**Proposed change:** Add an explicit "not a disqualifier" section to the Claygent prompt listing: maintenance mode pages, "we'll be back soon" with no other disqualifier signals, geolocation blocks ("this site is not available in your region"), and temporary HTTP errors where content was previously confirmed.

---

## 3. Login-only pages — consider `requires_auth` flag

Domains that show only a login form with no public content are currently Valid. This is correct — login-only pages belong to real business sites. However, downstream scraping agents will consistently fail to extract meaningful content from these domains and may produce empty or low-quality output.

**Why deferred:** `validate_domain`'s job is domain validity, not content extractability. A `requires_auth` signal is a different concern and may be better handled by the downstream scraping function that actually fails to extract content.

**Trigger to implement:** A named downstream function consistently fails or produces empty output on login-only domains, and there is a confirmed need to gate on this before running that scraper.

**Proposed change:** Add `requires_auth: boolean, optional output` (additive — no version bump). Claygent detects when a page shows only a login form with no public content and sets this to `true`. Callers can then choose to skip further scraping or route differently.
