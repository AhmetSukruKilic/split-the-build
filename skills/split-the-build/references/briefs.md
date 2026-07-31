# Personal Ledgers: file shapes, pointers, protocols

Each teammate gets `.split/{name}/` — a self-contained execution ledger in
initiative-ledger style. Templates in [../assets/](../assets/):
`state-template.md`, `execution-prompt-template.md`, `brief-template.md`,
`task-template.md`.

## The self-containment rule

A personal ledger passes if its owner can start a **fresh session**, paste
`EXECUTION_PROMPT.md`, and work a full task without reading anyone else's
folder or asking what to do next. Contracts are pasted into BRIEF.md, not
linked-only. Duplication across people's folders is fine — they are consumed,
not maintained.

## `STATE.md` — the personal task ledger

Uniform row schema regardless of where tasks come from:

```markdown
| ID | Title | Source | Depends on | Status |
|----|-------|--------|------------|--------|
| P03 | Webhook retry | `../../.billing/tasks/P03-webhook-retry.md` | P01 | todo |
| X01 | Uptime bars | `../../docs/plans/dashboard-plan.md#phase-2` | — | todo |
| G01 | Fix env validation | `tasks/G01-env-validation.md` | — | todo |
```

- **LDD-sourced** (`P03`): keep the source ledger's ID. The source task file is
  the single source of truth — the executing session ticks steps and writes
  `## Notes` **there**. Personal STATE.md mirrors status only.
- **Doc/list-sourced** (`X01`): generated ID, prefix `X`, numbered in
  assignment order. Pointer is file + heading anchor. Source doc is never
  edited; status lives only in personal STATE.md.
- **Generated** (`G01`): no source existed; task file written into this
  folder's `tasks/` from `task-template.md` — verbatim goal, `file:line`
  anchors, checkbox steps, and a literal runnable `## Verification` command.
  Never generate a task the person couldn't finish without asking a question.

Sections, in order: current task pointer → environment (literal commands) →
open blockers → task ledger → cut list reference. See the template.

### ID stability across re-runs

IDs are permanent addresses. On re-run: LDD IDs are already stable; `X`/`G`
items are matched to existing rows by source anchor (not by title) and keep
their IDs. New items get the next free number. Never renumber, never reuse.

## `EXECUTION_PROMPT.md`

Same protocol shape as plan-initiative's: read own STATE.md in full → pick the
current/first eligible task → **follow the pointer** and read the source task
file (or the generated one) → read BRIEF.md's files-owned and contracts
sections → do the work → verification verbatim → write Notes at the source →
update own STATE.md row → commit as `{ID}: <title>` → STOP, one task per
session. Plus two split-specific guardrails, verbatim in every prompt:

> - Never edit files outside the "Files you own" list in BRIEF.md.
> - Never edit a frozen contract; follow the change protocol in BRIEF.md.

## `BRIEF.md`

From `brief-template.md`. Notes per section:

- **Mission** — one sentence, outcome-phrased, not activity-phrased.
- **Files you own** — exact paths, including files the person will *create*.
  This is the git-conflict firewall.
- **Files you must not touch** — other slices' territory + single-owner shared
  files, each with who to ask instead.
- **Contracts** — *you provide* vs *you consume*, full definitions pasted in.
- **Hour-zero stubs you can rely on** — what already fakes it on main.
- **Definition of done** — concrete checks runnable without other slices
  finished. Not "it works".
- **Demo responsibility** — the sentence(s) of the final demo this slice owns.
- **If behind, cut in this order** — pre-agreed, so the person cuts scope
  alone at 2am without a meeting.

## Write-back rules

- **LDD ledgers only**: add an `Owner` column to the source `.{slug}/STATE.md`
  task table. That is the entire permitted edit to any source.
- Plan docs, TODO lists, READMEs: **never edited**. Their ownership mapping
  lives solely in personal STATE.md pointer rows.
- Unrecognized plan format: don't write to it, don't parse it silently — ask.

## Re-run protocol (document in SPLIT-PLAN.md)

`update-initiative` and daily work create new, ownerless tasks. Re-running
split-the-build:

1. Re-inventory sources; diff against existing `.split/` assignments.
2. Assign only new/unassigned items — existing assignments are stable unless
   the user asks to rebalance.
3. Append rows to personal STATE.md files; update Owner column for new LDD rows.
4. Re-check the no-file-owned-twice rule against the new items' footprints —
   a new task touching another slice's files goes to that slice's owner, or
   its shared surface becomes a new contract.

## Blocked-teammate protocol

Include verbatim in every BRIEF.md:

> **If you're blocked:**
> 1. **Blocked on another slice's unfinished work?** Don't wait — code
>    against the frozen contract and the hour-zero stub. Integration freeze
>    is when real implementations replace stubs, not before.
> 2. **Stub missing or broken?** That's a hair-on-fire bug for the stub's
>    owner. Ping them immediately; meanwhile hardcode a fake in *your own
>    files* and tag it `// TEMP-UNBLOCK` so it's greppable at merge time.
> 3. **Contract doesn't cover your case?** Do not silently extend it. Ping
>    provider + consumers; follow the change protocol. While waiting, build
>    the parts that don't depend on the gap.
> 4. **Timebox: 30 minutes.** Blocked longer with no answer → skip to your
>    next independent task and post what you skipped.
> 5. **Never fix a blocker by editing files you don't own.**
