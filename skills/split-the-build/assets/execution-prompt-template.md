# {{PERSON_NAME}} — {{SLICE_NAME}} — Execution Prompt

Paste this verbatim as the prompt for each new session working
`.split/{{name}}/`. One session = one task. The session has no memory of prior
sessions — everything needed lives in these files.

---

## Prompt (copy from here down)

You are executing one task from {{PERSON_NAME}}'s work package in
`.split/{{name}}/`. Follow this protocol exactly, in order. Do not skip steps,
do not batch tasks, do not improvise scope beyond the task.

1. **Read `.split/{{name}}/STATE.md` in full** — ledger, statuses, current-task
   pointer.
2. **Pick the task:** the "Current task" if named (the user's message wins if
   it names a different ID); otherwise the first `todo` row whose `Depends on`
   is empty or all-`done`. Nothing eligible? Stop and report — don't invent work.
3. **Read `.split/{{name}}/BRIEF.md`** — especially "Files you own", "Files you
   must NOT touch", and both contract sections.
4. **Follow the task's Source pointer** and read what it points to:
   - LDD task file (`../../.{{slug}}/tasks/…`): that file is the source of
     truth. Tick its checkboxes, write its `## Notes`, run its
     `## Verification` verbatim.
   - Plan-doc heading: read that section only. Verification and done-criteria
     are in the STATE.md row's task or BRIEF.md's definition of done.
   - Generated file (`tasks/…`): treat like an LDD task file.
5. **Do the work.** Stay inside "Files you own".
6. **Run the verification exactly as written.** Don't claim done without seeing
   it pass. If it fails, fix the code — never the command.
7. **Close out:** Notes at the source (LDD/generated); flip your STATE.md row;
   repoint "Current task"; rewrite "Last session ended".
8. **Commit** as `{{ID}}: <title>` on branch `{{BRANCH_NAME}}`.
9. **STOP.** The next task is the next session's job.

### Guardrails (apply regardless of task)

- Never edit files outside the "Files you own" list in BRIEF.md.
- Never edit a frozen contract; follow the change protocol in BRIEF.md.
- Blocked? Follow BRIEF.md's "If you're blocked" protocol — 30-minute timebox,
  then next independent task.
{{- Extra repo-specific guardrails, e.g. the initiative's invariant, restated
as a stop condition.}}
