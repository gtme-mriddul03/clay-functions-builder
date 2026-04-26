# Research

POV memos and disagreement map from the framework's founding design process.

## How we got here

The framework's rules weren't written top-down. We ran a structured research phase first:

1. **Four POV agents** (in `.claude/agents/`) each answered the same six design questions from a different lens:
   - **Engineer** — composability, primitives, interface contracts, debuggability
   - **Practitioner** — daily Clay use, waterfall patterns, Claygent, credit cost
   - **Writer** — documentation burden, discoverability, template usability
   - **Skeptic** — maintenance cost, sprawl, over-engineering, what to cut

2. **Each agent produced a memo** (800–1500 words) covering their lens's position on: when to build, scope design, output design, versioning, deprecation, and composition.

3. **We synthesized a disagreement map** — 13 disagreements identified across the four memos, ranked by whether downstream SOPs depend on their resolution (load-bearing vs peripheral). For each disagreement, we proposed a resolution with explicit justification.

4. **Mriddul approved the resolutions.** Those became the framework rules in `../skill/`, using Anthropic's skill-creator. 

The disagreement map is the single document that explains *why* every rule is what it is. Start there if a rule feels arbitrary.

## What's here

| File | What it is |
|---|---|
| `pov-memos/engineer.md` | Engineer lens: composability, primitives, interface contracts |
| `pov-memos/writer.md` | Writer lens: documentation, discoverability, template usability |
| `pov-memos/practitioner.md` | Practitioner lens: daily Clay use, waterfall patterns, Claygent |
| `pov-memos/skeptic.md` | Skeptic lens: maintenance cost, sprawl, over-engineering risks |
| `pov-memos/disagreement-map.md` | Synthesis: 13 disagreements, 10 load-bearing, approved resolutions |
| `.claude/agents/` | The four POV sub-agents — reusable for future research phases |

## Why this is in the same repo as the skill

Every rule in `../skill/` traces back to a specific disagreement in `disagreement-map.md`. Keeping research and skill together means a future maintainer can answer "why does this rule exist?" without leaving the repo.
