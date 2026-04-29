# Contributing

How we work on this repo as a two-person team.

---

## Repo structure recap

| Folder | What it is |
|---|---|
| `functions/` | Shipped Clay Functions — one subfolder per function |
| `skill/` | The clay-functions-v2 framework skill |
| `research/` | Historical POV memos — don't edit |

Functions and skill improvements are separate workstreams with separate PRs. Don't mix them.

---

## Branching

### Conventions

| Work type | Branch name |
|---|---|
| New function | `functions/<verb_noun>` e.g. `functions/validate-domain` |
| Function version bump | `functions/<verb_noun>-v2` e.g. `functions/validate-domain-v2` |
| Skill improvement | `skill/<short-description>` e.g. `skill/formula-column-template` |

### Rules

- Nobody pushes directly to `main`. Everything goes through a PR.
- Every PR requires **1 approval** from the other person before merging.
- Function PRs and skill PRs stay separate — different lifecycles, different revert implications.
- The only exception: a template fix so tightly coupled to a specific function that separating the diff would be incomprehensible. Note it explicitly in the PR description if you do this.

---

## Starting a new function

1. Cut a branch: `git checkout -b functions/<verb_noun>`
2. Open a new GitHub Issue for the function. Add it to the Projects board under **Backlog**.
3. Run `/clay-functions-v2` in your Claude Code session.
4. Move the Issue card to **In Design** on the board.
5. Work through the skill gates. Files go to `functions/<verb_noun>/`.
6. Move the card to **Building in Clay** once the spec is locked and you're building in Clay.
7. When published in Clay, open a PR from your branch → `main`.
8. Other person reviews and approves.
9. Merge. Move the card to **Published**.
10. Add a row to `functions/README.md` if not already there.

---

## Cutting a version bump

1. Cut a branch: `git checkout -b functions/<verb_noun>-v2`
2. Move the function's Issue card on the board from **Needs Version Bump** to **In Design**.
3. Run `/clay-functions-v2` and work through the version SOP (`skill/sop/02-version.md`).
4. Update `spec.md` change log and version increment section.
5. PR → `main`, 1 approval, merge.

---

## Improving the skill

1. Cut a branch: `git checkout -b skill/<short-description>`
2. Open a GitHub Issue if the improvement needs discussion. Otherwise, work from `skill/backlog.md` directly.
3. Make changes in `skill/sop/`, `skill/references/`, `skill/assets/templates/`, or `skill/backlog.md`.
4. PR → `main`, 1 approval, merge.
5. **After merging: sync the global skill installation on both machines** (see below).

---

## Syncing the skill after a skill PR merges

The skill is installed globally at `~/.claude/skills/clay-functions-v2/` on each person's machine. Changes merged to `main` don't sync automatically — both people need to update manually after every skill PR.

```bash
# From the repo root, after pulling main
cp -r skill/ ~/.claude/skills/clay-functions-v2/
```

Do this immediately after a skill PR merges. If you skip it, your local Claude sessions will run stale skill SOPs and templates — which means gate outputs and templates won't match the spec.

---

## GitHub Projects board

Lives on the repo under the **Projects** tab. One board, two workstreams (functions and skill) distinguished by labels.

### Columns

| Column | What lives here |
|---|---|
| Backlog | Planned functions + skill improvements not yet started |
| In Design | Function being scoped (skill session active) |
| Building in Clay | Spec locked, being built in Clay |
| Testing | Live in Clay, being validated on real data before full rollout |
| Published | Tested and rolled out to all consumers |
| Needs Version Bump | Caller change identified, v2 not yet started |
| Skill: In Progress | Skill improvement actively being worked |

### Labels

| Label | Applies to |
|---|---|
| `function` | Any Issue tracking a Clay Function |
| `skill` | Any Issue tracking a skill improvement |
| `breaking` | Version bump that requires a v2 |
| `additive` | Version bump that doesn't require a v2 |

### How to move cards

Move the card yourself when you change the stage. Don't wait for the other person to move it. If a card is in the wrong column it means the board is stale — fix it when you notice it.

---

## PR description format

Keep it short. Three things:

1. **What changed** — one sentence
2. **Files touched** — list them
3. **Board card** — link the Issue so GitHub auto-closes it on merge

---

## What goes in a PR vs. what stays in session notes

| Belongs in PR | Stays in session |
|---|---|
| Spec, canvas, usage, backlog files | Intermediate Socratic questions |
| ADR for a non-obvious decision | Draft outputs before the spec is locked |
| Skill SOP or template changes | Exploratory iterations on a prompt |
| README / register updates | |
