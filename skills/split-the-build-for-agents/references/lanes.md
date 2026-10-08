# Lanes: isolation, the per-task loop, merging, recovery

## Isolation

One row per lane, in the run ledger, filled **before** the first dispatch:

| Lane | Worktree | Branch | Backend port | Frontend port | DB | Test DB |
|------|----------|--------|--------------|---------------|----|---------|
| A | `.worktrees/<slug>-a` | `<slug>-a` | 8011 | 5181 | `<app>_<slug>_a` | `<app>_test_<slug>_a` |

Preflight, once:

1. **Integration branch + worktree** exist (the branch the PR will come from).
   Lanes branch from it, never from `main`.
2. **Ports free:** `lsof -tiTCP:<port> -sTCP:LISTEN` for every port in the table.
   If something holds one, find its owner (`lsof -a -p <pid> -d cwd`) — never kill
   another project's process; pick another port.
3. **Every resource overridable in code.** Grep for the default port, DB name and
   service URL in configs (dev-server proxy targets, e2e base URL, `.env`
   defaults, test fixtures). A literal with no env override is an hour-zero task:
   make it configurable, merge it, then fan out.
4. **Database per lane:** create the DB, run migrations and seed with the lane's
   env. Test databases per lane so test runs never collide.
5. **Installs per worktree** (venv, `node_modules`, git hooks). Generated build
   files a toolchain rewrites on every run (e.g. `tsconfig.tsbuildinfo`) are
   reverted before each commit — put that in every dispatch.
6. **Single-instance resources** — a browser extension, a device, a paid sandbox,
   a fixed webhook URL: serialize their use, or tell lanes to use the fallback
   (e.g. headless Playwright) and say so in their Notes.

## The per-task loop

Per lane:

1. Record the lane's BASE SHA. Dispatch the implementer (lane-dispatch template).
2. On DONE: check the report — commits exist on the lane branch, verification ran
   and passed, no edits outside owned files (`git diff --stat BASE..HEAD`). Then
   mark the task ready to merge. In the same message, dispatch the lane's next
   task if its deps are merged.
3. DONE_WITH_CONCERNS / BLOCKED / NEEDS_CONTEXT → answer or fix the brief and
   resume the same implementer; if it can't be unblocked, park the lane.

Never pick a model per task. Omit the model on every dispatch so each implementer
inherits the controller's configured model.

## Merging

Only the controller merges, into the integration worktree:

1. Merge order = dependency order; among peers, the lane that owns a hot file
   others queue behind goes first.
2. `git merge --no-ff <lane-branch>` (or the repo's convention). A conflict in a
   lane-owned file means the lane map was wrong: stop, fix the map, don't
   hand-resolve blind.
3. Regenerate controller-owned files (baselines, snapshots, lockfiles) with the
   repo's own generator; never hand-merge them. No generator? Take the lower
   value per key for shrink-only files, then run the check that reads them — it
   must pass with the merged result.
4. Update the plan's STATE/ledger rows and the run ledger (`<ledger-dir>/split-run/`)
   yourself, and commit them on integration with the merge — a fresh session reads
   both from the branch.
5. Run the fast suite (typecheck + unit) on integration. Red → the last merged
   lane owns the fix.
6. **Merge forward:** for each running lane that consumes what just landed,
   tell it to `git merge <integration-branch>` at the start of its next task (or
   between tasks). Never merge into a lane while its agent is mid-task.
7. Push integration when the repo's hooks allow (pre-push suites need a lane-free
   env: the integration worktree's own ports/DB).

## Recovery

- **Compaction or a new session:** read the run ledger at `<ledger-dir>/split-run/run-ledger.md`
  in the integration worktree and `git log` on each lane branch. A task
  with a `complete` line is done; never re-dispatch it.
- **Usage limit / API error mid-task:** the agent's edits are uncommitted in its
  worktree. Resume the same agent with "you were cut off; `git diff` shows where
  you were". Don't dispatch a fresh one over a dirty tree.
- **Lane stuck (BLOCKED):** park the lane; other lanes continue. If a later lane
  depends on it, that dependent waits — never route around the dependency.
