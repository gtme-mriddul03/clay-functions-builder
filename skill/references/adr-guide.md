# ADR Guide — Logging Design Decisions

ADR = Architecture Decision Record. In this framework, it means: a short written record of a non-obvious design call, who made it, when, and why.

You don't need an ADR for every decision. You need one when a future maintainer might look at the Function and wonder "why did they do it this way?" — and the right answer isn't obvious from the design.

---

## When to write an ADR

Write one when:

- You chose one Function over two (or vice versa) and the tradeoff wasn't obvious
- You included or excluded a reasoning string after thinking about it
- You chose an enum over a boolean because you anticipated more than two states
- You made a breaking change and the reason wasn't just "we added a field"
- You set a default value that seems wrong at first glance but is right for the use case
- You decided not to include a confidence score

Don't write one when:

- The decision is obvious from the spec
- You're just noting a bug fix
- You're adding a field with no tradeoff

---

## Where ADRs live

`decisions/YYYYMMDD-<function-name>-<short-slug>.md`

Example: `decisions/20240115-validate-domain-no-confidence-score.md`

---

## ADR format

Copy `assets/templates/adr-template.md`. Fill it. The whole thing should fit on one screen.

The template has three body sections plus a metadata header:

1. **Context** — what was the situation and what options were on the table?
2. **Decision** — what did we choose?
3. **Consequences** — what's better now, what's worse, what did we give up?

Plus a metadata block at the top of the file (Date, Function, Status). **Status** is `accepted` or `superseded by YYYYMMDD-<slug>` and lives in the preamble — not as its own `## Status` heading.

If a later decision supersedes this one, update the Status field in the preamble and link to the newer ADR. Don't delete the old one.

---

## ADR log

Keep a running table in `decisions/README.md`. One row per ADR:

| Date | Function | Slug | Status |
|---|---|---|---|
| 2024-01-15 | validate_domain | no-confidence-score | accepted |

This is the only place you need to look to find all decisions. The individual ADR files have the detail.
