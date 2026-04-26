# Clay Column Naming Conventions

Apply these when documenting Clay column names in specs and usage docs.

| Pattern | Convention | Example |
|---|---|---|
| General | Title Case — every word capitalized | Write To Serper Cache - 1 |
| Boolean / checkbox | Is [X]? | Is Verified?, Is ICP?, Is LinkedIn URL Valid? |
| CRM writeback output | (o) prefix | (o) Most Frequent Number |
| Route-row (write to table) | Route: Table Name | Route: Enrichment Mainframe |
| Lookup (read from table) | Lookup: Table Name | Lookup: Serper Cache - 1 |
| Normalize — action column | Normalize [Field] | Normalize Phone |
| Normalize — clean output | Normalized [Field] | Normalized Phone |
| LLM agent (no internet) | LLM: [Verb] [Subject] | LLM: Extract Top 3 Contacts |
| Claygent (web research) | Claygent: [Verb] [Subject] | Claygent: Find LinkedIn URL |
| Email sequencer variable | Variable: var_name | Variable: hq_country |
| Stage keywords | Interim / Combined / Final | Interim Score, Combined Phone, Final ICP Score |
| Large array split (>8KB) | [Name] - Core / [Name] - Text | All Field Candidates - Core |
| Numbered variants | Hyphen + number suffix | Lookup: Serper Cache - 2 |
| Source columns | Default Clay naming | Rows from: Enrichment Mainframe |
| Function call column | Title Case matching the Function's purpose | ICP Industry Check, Domain Validator |

## Applying these to Function outputs

When filling the Clay column names section of a spec, map each output field:

- Boolean outputs → `Is [X]?`
- Claygent fetch steps → `Claygent: [Verb] [Subject]`
- LLM classification steps → `LLM: [Verb] [Subject]`
- Final scored/classified output → `Final [X]` or `Interim [X]` depending on position in pipeline
- Pass-through description outputs → `Claygent: Find [X]` (matches the agent that produced it)
