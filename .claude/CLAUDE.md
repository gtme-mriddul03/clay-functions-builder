# Clay Functions Framework — Project Context

## Who drives this repo

Mriddul is a GTM Engineer and CRM data specialist who hosts Clay Club Delhi. He works daily in Clay, Claygent, waterfall enrichment, scoring engines, and N8N round-trips.

## What this repo is

A framework for designing, documenting, versioning, and governing [Clay Functions](https://www.clay.com/functions). Clay Functions are reusable enrichment workflows with defined inputs, a workflow body, structured outputs, referenced as a single column in any Clay table.

## Repo structure

| Folder | What it is |
|---|---|
| `research/` | POV memos and disagreement map that founded the v2 skill's design rules. Historical — don't edit. |
| `skill/` | The v2 Clay Functions skill. Invoke `/clay-functions-v2` to use it. PRs here = framework improvements. |
| `functions/` | Live Clay Functions. Each function gets its own subdirectory. |

## Agents

`research/.claude/agents/` contains the four POV agents (engineer, writer, practitioner, skeptic) used to generate the founding research in `research/pov-memos/`. They are available if you need to re-run the research phase for future framework decisions.

## House conventions

- **Markdown only.** GFM. No HTML. Relative links between files must resolve.
- **Voice:** first-person plural for conventions we're adopting; second-person for instructions the reader acts on.
- **Tone:** casual but crisp. Direct. No corporate hedging.
- **No implementation of actual Clay table logic** in this repo. Structure, conventions, and function docs only.

## Pointer table

| Working on... | Go to |
|---|---|
| Building a new Function | `functions/` — invoke `/clay-functions-v2` first |
| Improving the framework | `skill/` |
| Understanding why the rules are what they are | `research/pov-memos/disagreement-map.md` |
| Onboarding a new hire | This file → `research/` → `skill/` |
