# SOP 04 — Pre-Publish Checklist

Run this before publishing any Function. Render as a pass/fail table — one row per check, no prose.

| Item | Status | Note |
|---|---|---|
| **Interface** | | |
| Function name follows `verb_noun_vN` in Clay UI | | |
| Every required input documented with type | | |
| Every optional input has a documented default and defined behavior | | |
| Every output field has type, success value, and failure value | | |
| No output field is "TBD" or "varies" | | |
| Outputs flat, or one nesting level with 3+ related fields | | |
| `reasoning` field present only if a downstream step branches on its content | | |
| `confidence` field present only if a downstream threshold gate reads it | | |
| **Scope** | | |
| One-sentence description passes: verb-first, no "and", clear input→output | | |
| Sub-Functions (if any) are already published and versioned | | |
| **Documentation** | | |
| `functions/<name>/spec.md` exists and all sections filled | | |
| `functions/<name>/usage.md` exists with at least one real output example | | |
| Clay column names section filled in spec | | |
| Version increment triggers documented in spec | | |
| Change log has at least the initial v1 entry | | |
| Non-obvious design decisions noted as ADRs in spec | | |
| **Versioning** | | |
| v1: spec shows version 1, change log has initial entry | | |
| v2+: previous version still live | | |
| v2+: consumer migration window calendared | | |
| **Consumers** | | |
| All consuming tables listed in spec | | |
| Breaking change: each consumer confirmed for update within migration window | | |

---

## Publish

Only publish after every applicable row is Pass. Clay's sandbox diff shows what changes — read it before confirming.

## After publish

- Update spec change log with publish date
- If breaking: calendar the migration deadline
- Notify owners of downstream tables
