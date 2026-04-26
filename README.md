# Clay Functions Framework

Framework for designing, versioning, documenting, and governing [Clay Functions](https://www.clay.com/functions).

## What's in this repo

| Folder | What it is |
|---|---|
| [`research/`](research/) | Founding POV memos and disagreement map — the audit trail behind the framework's rules |
| [`skill/`](skill/) | v2 Clay Functions skill — SOPs, templates, references, ADR log |
| [`functions/`](functions/) | Clay Functions built using the framework |

## Reading order

New here? Start with [`research/pov-memos/disagreement-map.md`](research/pov-memos/disagreement-map.md) to understand why the rules are what they are, then read [`skill/SKILL.md`](skill/SKILL.md) for the framework itself.

Building a function? Go straight to [`functions/`](functions/).

## Using the skill

The framework is packaged as a Claude Code skill. Install the contents of `skill/` into `~/.claude/skills/clay-functions-v2/` and invoke `/clay-functions-v2` in any conversation.
