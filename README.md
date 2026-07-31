# split-the-build

A Claude Code skill that splits an existing codebase into **per-person work
packages** so a team can build in parallel without blocking each other —
hackathons, sprint kickoffs, crunch integration.

Companion to [ledger-driven-development-skill](https://github.com/sezaiemrekonuk/ledger-driven-development-skill):
when initiative ledgers (`.{slug}/` folders from `plan-initiative`) or other
plan artifacts exist, this skill consumes them and divides their tasks across
teammates. When nothing exists, it maps the code and generates the tasks itself.
Either way, each teammate gets an initiative-style execution ledger they can
work from in a fresh session.

## What it does

1. **MAP** — inventories the repo (entry points, modules, data models, external
   services, working-vs-stubbed) and every work source: LDD ledgers, plan docs,
   TODO lists. Stops and shows you the map before slicing.
2. **SLICE** — cuts work into ownership slices defined by **files, not
   features**. No file is owned twice; shared files become contracts.
3. **CONTRACTS** — freezes every cross-slice interface as real code (REST,
   GraphQL, DB schema, events, env vars) before anyone starts.
4. **STUB FIRST** — an hour-zero checklist of stubs and fixtures committed to
   main so nobody waits on anyone.
5. **LEDGERS** — one execution ledger per person in `.split/`.
6. **MERGE PLAN** — branch strategy, merge order, integration freeze, cut list,
   riskiest dependency + fallback.

## Output

```
.split/
  SPLIT-PLAN.md            # map, slices, contracts, hour-zero checklist, merge plan
  {name}/                  # one per teammate
    STATE.md               # personal task ledger — pointer rows into sources
    EXECUTION_PROMPT.md    # paste into a fresh session, zero prior context
    BRIEF.md               # files owned/forbidden, contracts, stubs, cut list
    tasks/                 # only for generated tasks (no plan source existed)
```

Source task files are **pointed to, never copied or moved** — the source ledger
stays the single source of truth, and `update-initiative` keeps working
untouched. The only write-back is an `Owner` column added to LDD ledgers'
STATE.md tables.

## Install

```bash
git clone https://github.com/{you}/split-the-build.git
cp -r split-the-build/skills/split-the-build ~/.claude/skills/
```

Or per-project: copy (or symlink) into the repo's `.claude/skills/`.

## Use

In Claude Code, in the repo you want to divide:

> split this across the team

Also triggers on: "divide the work", "who should build what", "parallelize this
project", "work breakdown", "assign modules", "split the ledger".

The skill will ask for team size/names/strengths, time remaining, and the one
thing that must be demoable — then map, stop for your confirmation, and build
`.split/`.

## Works best with planning skills

split-the-build is a divider, not a planner — it shines when a planning skill
has already produced structured work:

- **[ledger-driven-development-skill](https://github.com/sezaiemrekonuk/ledger-driven-development-skill)**
  (`plan-initiative` / `update-initiative`) — the primary companion. Plan
  initiatives into `.{slug}/` ledgers first; split-the-build then divides
  their tasks across teammates by file footprint, adds an `Owner` column, and
  leaves the ledgers as the single source of truth. Amendments made later with
  `update-initiative` are picked up by re-running the split.
- **Plan docs** — implementation plans in `docs/plans/*.md` or specs with
  phase/task headings (e.g. from a writing-plans style skill) are consumed
  read-only via heading-anchor pointers.
- **Task lists** — `TODO.md` and checkbox lists in README/CLAUDE.md count too.

No planning skill in play? Still works: it maps the code and generates the
task files itself.

## Principles

- File ownership, not feature ownership — files are what conflict in git.
- Fewer fat slices beat many thin ones — coordination cost is real.
- If something can't be parallelized, it goes on one person's critical path —
  never fake a split.
- Existing plans are the source of truth — pointers, not copies.
- Every assumption made from an ambiguous repo is surfaced, never silent.

## Layout

```
skills/split-the-build/
  SKILL.md                 # the skill
  references/contracts.md  # contract formats per stack + change protocol
  references/briefs.md     # personal-ledger file shapes, pointer rules, re-run protocol
  assets/                  # templates: BRIEF, STATE, EXECUTION_PROMPT, generated task
```
