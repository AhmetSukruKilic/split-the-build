# Lane dispatch prompt (one task)

Fill and send as the implementer's prompt. Keep exact values in the task file; this
prompt carries only what the task file can't know.

```
You are implementing task {{ID}} ({{title}}) in lane {{LANE}} of a parallel run.
Other agents are working in other worktrees at the same time — stay inside yours.

Read first: {{TASK_FILE}} — your requirements, exact values verbatim.
Then only what it tells you to read.

Your lane:
- Worktree: {{WORKTREE}} (branch {{BRANCH}}). Run every command there.
- Ports: backend {{PORT_BE}}, frontend {{PORT_FE}}. DB: {{DB}}; test DB: {{TEST_DB}}.
  Env to export: {{ENV_LINES}}
- Files you own: {{OWNED}}. Edit nothing else.
- Never edit: {{CONTROLLER_OWNED}} (the controller updates them at merge),
  other lanes' files, the integration branch.
- Contracts you consume / provide (FROZEN — don't change, report if they don't fit):
  {{CONTRACTS}}
- Earlier work you build on: {{INTERFACES_FROM_MERGED_TASKS}}
- {{MERGE_FORWARD: "Before starting, run `git merge <integration>`" | omit}}

Do the task: tick its steps and fill its Notes in the task file, run its
verification exactly as written, commit as `{{ID}}: <title>`
({{REVERT_GENERATED_FILES}} first). Skip the plan's own STATE/ledger updates,
push and any other end-of-task protocol steps — the controller does them at
merge. Stop any servers you started.

Write the full report to {{REPORT_FILE}} (`<ledger-dir>/split-run/{{ID}}-report.md` in
your worktree; do not commit it — the controller copies it into integration at merge). Reply only: Status (DONE |
DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT), commits, one-line test summary,
concerns, report path.
```
