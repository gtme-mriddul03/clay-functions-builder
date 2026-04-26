# Usage — `<function_name>_v<N>`

<!-- This is for Clay table builders. Fill it after your first real run with real output. -->
<!-- Keep it short. Link to the spec for interface details. -->

---

## What this does

<!-- One plain-language sentence. Not the spec's one-liner — simpler. -->
<!-- e.g., "Tells you whether a company domain is actually live and reachable." -->

---

## When to use it

<!-- 2–3 concrete use cases. Start each with "Use when" or a scenario. -->

- <!-- e.g., Use when you have a raw domain column from a CRM import and need to filter out dead companies before enrichment. -->
- <!-- e.g., Use when Claygent is hitting phantom domains and returning hallucinated company data. -->
- <!-- add a third if you have one -->

---

## Inputs

<!-- Only list inputs the table builder will actually configure. -->
<!-- If an input has a sensible default that most people won't touch, say so. -->

| Field | What to pass | Example |
|---|---|---|
| <!-- e.g., `url` --> | <!-- e.g., Raw domain or URL string from your column --> | <!-- e.g., `acme.com` --> |
| <!-- e.g., `depth` --> | <!-- e.g., `basic` for HTTP check only; `full` to also detect parked domains --> | <!-- e.g., `basic` --> |

---

## Output example

<!-- Paste a real output object from an actual run. Not a made-up example. -->
<!-- Show what it looks like in a Clay cell. -->

```json
{
  "<field>": "<value>",
  "<field>": "<value>"
}
```

<!-- e.g.:
{
  "is_valid": true,
  "status": "live",
  "canonical_url": "https://acme.com",
  "reasoning": "Domain resolves via HTTPS with no redirect to a different registrable domain."
}
-->

<!-- Failure example too: -->

```json
{
  "<field>": "<value>",
  "<field>": "<value>"
}
```

<!-- e.g.:
{
  "is_valid": false,
  "status": "parked",
  "canonical_url": null,
  "reasoning": "Domain redirects to a parking page at sedo.com."
}
-->

---

## Common patterns

<!-- 2–3 concrete formulas or downstream patterns that consume this Function's output. -->
<!-- These must be formulas someone can copy and actually use. -->

**<!-- e.g., Filter to live domains only: -->**
```
<!-- e.g., {function_name}.is_valid = true -->
```

**<!-- e.g., Branch in Claygent based on status: -->**
```
<!-- e.g., Only visit {{domain}} if {{function_name.status}} = "live" -->
```

**<!-- e.g., Enrich canonical URL, not the raw input: -->**
```
<!-- e.g., Use {{function_name.canonical_url}} as the URL input for downstream enrichment -->
```

---

## Full interface

See `docs/<function-name>-spec.md` for complete input/output types, defaults, and version history.
