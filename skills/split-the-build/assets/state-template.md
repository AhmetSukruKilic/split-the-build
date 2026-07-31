# {{PERSON_NAME}} — {{SLICE_NAME}} — State

Last updated: {{YYYY-MM-DD}}
Last session ended: {{empty at creation; executing sessions rewrite this — what
landed, which files, verification output, what the next task must know.}}

## Current task

**{{ID}} — {{title}}**
{{1–2 sentences: why this one is next and the one trap in it.}}

## Environment

{{What is already set up and what a session must start itself. Literal
commands — e.g. `docker compose -f docker-compose.dev.yml up`.}}

## Open blockers

{{Empty unless blocked. Each entry: what's needed, who owns it, which task it
unblocks. See BRIEF.md "If you're blocked" before adding anything here.}}

## Task ledger

Statuses: todo → in_progress → done (blocked if waiting).
Source column: LDD task file (ID kept, Notes/status live THERE) · plan-doc
heading anchor (X-prefixed ID, status lives here) · generated file in
`tasks/` (G-prefixed ID).

| ID | Title | Source | Depends on | Status |
|----|-------|--------|------------|--------|
| {{P03}} | {{title}} | `../../.{{slug}}/tasks/{{P03-slug}}.md` | {{IDs or —}} | todo |
| {{X01}} | {{title}} | `../../docs/plans/{{plan}}.md#{{heading}}` | — | todo |
| {{G01}} | {{title}} | `tasks/{{G01-slug}}.md` | — | todo |

## Cut order

If behind, cut from the bottom of this ledger per BRIEF.md's "If behind" list.
Never cut: {{the task(s) feeding the one demoable thing}}.
