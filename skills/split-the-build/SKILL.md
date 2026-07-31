---
name: split-the-build
description: Splits an existing codebase into per-person work packages with
  file ownership, frozen interface contracts, stub-first ordering, and a merge
  plan. Consumes existing plans when present — initiative ledgers (.slug/
  folders), plan docs, TODO lists — and emits one execution ledger per teammate
  (pointer STATE.md, EXECUTION_PROMPT.md, BRIEF.md in .split/). Use when a team
  needs to divide a project across members to work in parallel — hackathons,
  sprint kickoffs, crunch integration. Triggers on "split this across the
  team", "divide the work", "who should build what", "parallelize this
  project", "work breakdown", "assign modules", "split the ledger".
---

# Split the Build

Divide an existing codebase — and its existing plans, if any — into N parallel
work packages, one per human teammate, such that nobody blocks anybody and the
merge at the end is boring. Output is one initiative-style ledger per person in
`.split/`, executable from a fresh session.

## Required inputs — ask if missing

1. **Team**: size, names, and each person's stated strengths (frontend, DB, infra, etc.).
2. **Time remaining** until the deadline or demo.
3. **The one thing that must be demoable** at the end. Everything else is negotiable.

If any are missing from the conversation, ask. Do not invent a team.

## Principles (non-negotiable)

- **File ownership, not feature ownership.** Files are what conflict in git.
  A slice is defined by the files and directories it owns, never by a feature
  description alone.
- **No file is owned twice.** If two slices need to edit the same file, that
  file becomes a contract (step 3) with a single owner, or gets split.
- **Prefer fewer fat slices over many thin ones.** Coordination cost is real.
  Three people with clear walls beat five people negotiating boundaries.
- **Don't fake a split.** If something genuinely cannot be parallelized, say so
  explicitly and put it on one person's critical path. A fake split costs more
  than an honest queue.
- **Existing plans are the source of truth.** Never copy or move a source task
  file; point to it. Never edit a plan format you don't recognize.
- **Surface every assumption.** Where the repo or a plan was ambiguous, list
  the assumption you made. Wrong silent assumptions become merge-day disasters.

## Workflow

### 1. MAP — inventory the repo and its plans

Explore the actual code. Never guess from file names; open the files. Inventory:

- Entry points, modules, data models, external services, build/deploy path
- **Working vs stubbed**: mark each area WORKS (verified in code), STUBBED
  (placeholder/TODO), or MISSING
- **Work sources**, in priority order:

| Source | Detect by |
|--------|-----------|
| Initiative ledger | `.{slug}/` folder with `STATE.md` + `tasks/` |
| Plan docs | `docs/plans/*.md`, `docs/superpowers/specs/*` with phase/task headings |
| Loose task lists | `TODO.md`, checkbox lists in README/CLAUDE.md |
| None | remaining work inferred from the code map |

Sources coexist — build one combined work-item inventory, each item labeled
with its source. Unrecognized plan structure? Report what you found and ask
whether its headings count as tasks. Never guess silently.

**STOP HERE.** Show the MAP (including the work-source inventory) to the user
and get confirmation before slicing. The user knows things the repo doesn't say.

### 2. SLICE — cut work into ownership slices

Cut into N slices, N = team size (or fewer — see principles). When work items
come from sources, the slice unit is the task and the constraint is each task's
file footprint (from its anchors): tasks touching the same files go to the same
person, or the shared file becomes a contract. For each slice define:

- **Owns**: exact file paths and directories. Every file that will change must
  belong to exactly one slice.
- **Shared files**: one owner; others request changes, or the file's interface
  becomes a contract.
- **Estimated hours** vs time remaining; balance load and match strengths.
- **Independent test**: how this person verifies their slice without anyone
  else's slice being done.

### 3. CONTRACTS — freeze the interfaces

Every point where one slice calls, imports, or reads another slice's work gets
a written contract **before work starts**. Contracts are real code — type
definitions, endpoint signatures, schema DDL, event payloads — not prose. Mark
each `FROZEN`. Changing one requires agreement from every consumer.

Read [references/contracts.md](references/contracts.md) for formats per stack
(REST, GraphQL, DB schema, events, env vars) and the change protocol.

### 4. STUB FIRST — hour zero checklist

Ordered checklist of stubs, mocks, and fixtures committed to main before anyone
starts: fake API responses, seeded data, no-op implementations behind real
interfaces, hardcoded auth. Goal: everyone runs the whole app locally at hour
zero and codes against stubs instead of waiting. Order by who unblocks whom.

### 5. LEDGERS — one execution ledger per person

Write `.split/` at repo root:

```
.split/
  SPLIT-PLAN.md            # map, slices, contracts, hour-zero checklist, merge plan
  {name}/
    STATE.md               # personal task ledger — pointer rows into sources
    EXECUTION_PROMPT.md    # pasteable into a fresh session
    BRIEF.md               # files owned/forbidden, contracts, stubs, cut list
    tasks/                 # only when an item has no source — generated task files
```

Pointer rows reference source task files/headings; ledger-sourced Notes and
status stay in the source file. Add an `Owner` column to source ledgers'
STATE.md tables — the only write-back allowed, and only for `.{slug}/` ledgers.
Each person must be able to work from their folder alone, in a fresh session.

Read [references/briefs.md](references/briefs.md) for file shapes, pointer
formats, ID stability, the blocked-teammate protocol, and the re-run protocol.
Templates: [assets/](assets/).

### 6. MERGE PLAN

End SPLIT-PLAN.md with:

- Branch strategy (branch per slice, naming, where stubs live)
- Merge order and why (who lands first, who rebases on whom)
- Integration freeze time: feature work stops, integration fixes only
  (rule of thumb: last 20–25% of remaining time)
- Cut list: what gets dropped, in order, if behind — protecting the one
  demoable thing
- Riskiest cross-slice dependency and its concrete fallback
