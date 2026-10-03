# split-the-build

A Claude Code skill that splits an existing codebase into **per-person work
packages** so a team can build in parallel without blocking each other —
hackathons, sprint kickoffs, crunch integration.

Works standalone or on top of any planning skill: when plan artifacts exist —
initiative ledgers, plan docs, task lists — this skill consumes them and
divides their tasks across teammates. When nothing exists, it maps the code
and generates the tasks itself. Either way, each teammate gets an
initiative-style execution ledger they can work from in a fresh session.

The repo also ships **[split-the-build-for-agents](#split-the-build-for-agents)**:
the same idea for a single Claude Code session that runs a plan's tasks as
parallel lanes of subagents instead of dividing them across people.

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
cp -r split-the-build/skills/split-the-build-for-agents ~/.claude/skills/   # optional
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

split-the-build is a divider, not a planner — it shines when any planning
skill has already produced structured work. It is not tied to a specific one;
anything enumerable into tasks counts:

- **Initiative ledgers** — `.{slug}/` folders with `STATE.md` + `tasks/`.
  Tasks are divided by file footprint, an `Owner` column is added, and the
  ledger stays the single source of truth; later amendments are picked up by
  re-running the split. One suggestion that produces this shape:
  [ledger-driven-development-skill](https://github.com/sezaiemrekonuk/ledger-driven-development-skill).
- **Plan docs** — implementation plans in `docs/plans/*.md` or specs with
  phase/task headings, consumed read-only via heading-anchor pointers.
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

## split-the-build-for-agents

For when the "team" is a set of subagents one controller session dispatches.
Serial subagent runs waste hours when tasks only depend on a shared foundation;
this skill runs them as **lanes**, each in its own git worktree, one implementer
per task.

1. **GRAPH** — dependencies plus each task's real file footprint (anchors + grep).
2. **LANES** — chains of tasks cut by files; no file edited by two lanes at the
   same time. Hot files queue behind one lane's merge; tasks split at a file
   boundary when only part is hot. Stops for your confirmation.
3. **ISOLATE** — per-lane worktree, installs, ports, database and test database,
   with a check that each is actually overridable in code.
4. **CONTRACTS + HOUR ZERO** — frozen cross-lane surfaces and the foundation
   tasks every lane needs, merged first.
5. **RUN** — lanes in parallel (cap 3–4), each lane's next task dispatched as
   soon as its deps merge, a git-ignored run ledger for compaction and
   usage-limit recovery.
6. **MERGE** — each task into the integration branch as soon as it reports DONE
   with verification passing; controller regenerates baselines/lockfiles and
   merges forward.
7. **CLOSE** — serial close-out.

Triggers on "parallelize the ledger for agents",
"run tasks in parallel with subagents", "split the build for agents".

## Layout

```
skills/split-the-build/
  SKILL.md                 # the skill
  references/contracts.md  # contract formats per stack + change protocol
  references/briefs.md     # personal-ledger file shapes, pointer rules, re-run protocol
  assets/                  # templates: BRIEF, STATE, EXECUTION_PROMPT, generated task
skills/split-the-build-for-agents/
  SKILL.md                 # the skill
  references/lanes.md      # isolation preflight, per-task loop, merging, recovery
  assets/                  # templates: run ledger, lane dispatch prompt
```
