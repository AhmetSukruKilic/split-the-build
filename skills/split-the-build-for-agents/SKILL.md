---
name: split-the-build-for-agents
description: Use when a multi-task plan or initiative ledger is about to be executed
  by AI subagents in one session and the tasks are not one strict chain — some
  depend only on a shared foundation, so running them one at a time wastes hours.
  Also use when a serial subagent run was too slow, when tasks "don't all depend on
  each other", or on "parallelize the ledger for agents", "run tasks in parallel
  with subagents", "split the build for agents", "parallel lanes", "fan out the tasks".
---

# Split the Build for Agents

Run a plan's tasks as **parallel lanes of subagents**, each lane in its own git
worktree, while keeping an implementer + reviewer loop per task. The human
`split-the-build` divides work across people; this divides it across concurrent
agents that one controller (you) dispatches, reviews, and merges.

**Core principle:** parallel agents are safe only when nothing they touch at the
same time is shared — not files, not the checkout, not ports, not the database.
Speed comes from removing waits, never from accepting conflicts.

**REQUIRED SUB-SKILL:** the per-task loop (implementer → reviewer → fix rounds →
re-review) is superpowers:subagent-driven-development, if installed. This skill
overrides only its "never dispatch implementers in parallel" rule — that rule
exists because of a shared checkout, which lanes remove. Without that skill, use
the loop in [references/lanes.md](references/lanes.md#the-per-task-loop).

## When NOT to split

- Fewer than 4 tasks, or the dependency graph is one chain → run serially.
- Every task edits the same few files → an honest queue beats a fake split.
- The machine can't isolate a second copy (step 3 fails and can't be fixed in an
  hour-zero task) → serial.

Say so and stop. A fake split costs more than an honest queue.

## Workflow

### 1. GRAPH — dependencies and file footprints

From the plan: each task's `Depends on`, plus its **file footprint** — every file it
will edit or create. Start from the task's `## Files touched` section when it has
one, then confirm with its anchors *and* a grep for what it migrates (call sites
hide in files the task file never names). Build a task × file matrix. Read the
files; never guess footprints from titles. A `Depends on` that is only about order
(no code dependency) is not a dependency — ask the user before treating it as one.

### 2. LANES — cut by files, not by features

A **lane** is a chain of tasks run by one agent at a time in one worktree.
Rules:

- **No file is edited by two lanes that run at the same time.** Overlap means:
  same lane, or serialize them (lane B branches after lane A merges), or turn the
  file into a contract (step 4). "Resolve the merge by hand later" is not an option.
- **Split a task at a file boundary** when only part of it is hot: the
  non-overlapping part runs in parallel, the hot-file part queues behind the
  other lane's merge. Record the split in the run ledger; the source task keeps
  its ID.
- **Controller-owned files** — the plan's STATE/ledger, generated baselines,
  snapshots, lockfiles: lanes never write them. You regenerate or edit them at
  merge time.
- **Additive registries** (barrel `index.ts`, shared CSS, shared test helpers,
  route tables): one owner, or pre-cut one section per lane at hour zero.
- Cap concurrency at **3–4 implementers**. More hits usage limits and machine
  load before it saves time.

**STOP.** Show the user the graph, the lanes, the matrix hot spots, and every
assumption. Get confirmation before creating anything.

### 3. ISOLATE — preflight every shared resource

Per lane: a worktree branched from the **integration branch**, its own dependency
installs, ports, database / test database, and a clean env. Verify each is
overridable by env/flag *in the code* (a hardcoded proxy target or DB name is a
blocker). If one isn't, making it configurable is an hour-zero task. Single-instance
resources (one browser extension, one device) get serialized and named in briefs.
Table and commands: [references/lanes.md](references/lanes.md#isolation).

### 4. CONTRACTS + HOUR ZERO

Write every cross-lane surface as real code before dispatch, marked `FROZEN`:
component props, exported names, CSS class names, test-helper signatures, env
vars. Hour zero = only the tasks **every** lane needs, plus the stubs, config
fixes from step 3 and section cuts lanes code against, merged into the integration
branch **before** any lane starts. A task only some lanes need is the head of a
lane, not hour zero. Contract
formats: the `split-the-build` skill's `references/contracts.md`, if installed.

### 5. RUN — lanes in parallel, reviews pipelined

- Write the run ledger first ([assets/run-ledger-template.md](assets/run-ledger-template.md))
  at `<integration-worktree>/.split-agents/run-ledger.md`, git-ignored through
  `.git/info/exclude`; reports and review packages go beside it. After compaction
  or a usage-limit stop, trust it and `git log`, not memory.
- Dispatch each ready lane's next task with
  [assets/lane-dispatch-template.md](assets/lane-dispatch-template.md): one task,
  its worktree, its ports/DB, its owned files, the contracts, the report path.
- **Start a task the moment its dependencies are merged**, not at a wave boundary.
- **Pipeline reviews:** when a lane's task reports DONE, dispatch its reviewer
  (read-only on that SHA) *and* the lane's next independent task together. A fix
  round goes to the same lane after its in-flight task commits.
- Lane agents commit and stay on their branch; they never push, never touch the
  integration branch, never edit controller-owned files.
- **The plan's per-task protocol is split in two.** If its tasks say "update
  STATE.md, push, name the terminal, stop": the lane agent does the task-file
  part (tick steps, fill Notes, run verification, commit); you do the rest
  (STATE rows, push) at merge time. Say this in every dispatch.

### 6. MERGE — continuously, in dependency order

Merge a lane task into the integration branch as soon as its review is clean —
never batch everything for the end. After each merge: regenerate controller-owned
files, run the fast suite on integration, then merge integration forward into any
running lane that consumes what just landed. Protocol:
[references/lanes.md](references/lanes.md#merging).

### 7. CLOSE — one final review, serial close-out

Once every lane is merged: the whole-branch review (most capable model) on the
integration branch, one fix wave, then close-out tasks (cross-browser e2e,
archive, PR) serially in the integration worktree. Remove lane worktrees, drop lane
databases, stop lane servers. Keep the branches unless told otherwise.

## Red flags — stop

| Thought | Reality |
|---------|---------|
| "Two agents in one checkout is fine, different files" | Shared index, build caches, installs, ports. One lane = one worktree. |
| "Small overlap, I'll resolve the conflict" | Conflicts in parallel lanes surface late and get resolved blind. Serialize or contract. |
| "Skip reviews, we're parallel now" | Speed comes from overlap, not from dropping gates. |
| "Let each lane update STATE.md" | Controller-owned. Lanes report; you write. |
| "Merge all lanes at the end" | Big-bang merge. Merge each task when its review is clean. |
| "Ports/DB probably configurable" | Check the code. Hardcoded = hour-zero task. |
| "8 lanes = 8× faster" | Usage limits and CPU. Cap 3–4. |

## Example — a 9-task UI ledger

```
W01 ─┬─ W02 ─┬─ W03 ─┐        Lanes: hour zero W01 · A W02→W03→W04 ·
     │       └─ W04 ─┤        B W05 · C W06 · D W07 · close W08→W09
     ├─ W05 ─────────┼─ W08 ─ W09
     ├─ W06 ─────────┤        Hot files: ChargesPanel, ShelfPanel, LedgerPage,
     └─ W07 ─────────┘        AdminPage (edited by A, B and C)
```

B and C build their components and migrate their non-hot callers in parallel
with A. Their hot-file migrations are split off and queue behind A's W04 merge,
then run B → C. `ui/index.ts` and `ui.css` get one pre-cut section per lane at
hour zero. `baseline.json` and the ledger's STATE.md are controller-owned.

Serial run: ~5 h. Lanes + pipelined reviews: ~2.5–3 h, same agents, same reviews.
