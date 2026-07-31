# {{ID}} — {{Title}}
Depends: {{IDs or —}} · Status: todo
Read first: ../STATE.md, ../BRIEF.md, then this.

## Goal

Owner's ask:

> {{The user's/team's own words, verbatim. Never paraphrase — paraphrase is
> how scope drifts.}}

{{2–3 sentences translating it into this task's slice. Name the teammate who
owns the adjacent half, if any.}}

## Non-negotiables

- {{Constraint stated as a prohibition, with the reason.}}
- Stay inside BRIEF.md's "Files you own".

## Context (anchors)

- `{{path/to/file}}:{{line}}` — {{what lives here and why this task cares}}.
- {{The trap: sibling call sites, a contract this touches, an assumption from
  SPLIT-PLAN.md.}}

## Steps

- [ ] {{One action. Concrete enough to do without a decision.}}
- [ ] {{…}}
- [ ] Tests: {{each case, named, including the negative case}}.

## Definition of done

- {{Observable outcome, stated from outside the code.}}
- {{The thing that must remain unchanged.}}

## Verification

`{{literal runnable command}}`

{{Then manual steps, in order, with expected results.}}

## Notes

{{Empty until done. The executing session fills this with: what actually
happened, deviations, verification output verbatim, and a hand-off paragraph
for the dependent task/teammate.}}
