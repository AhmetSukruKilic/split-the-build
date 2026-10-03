# Run ledger — plan: {{PLAN_PATH}}

Integration: `{{INTEGRATION_BRANCH}}` in `{{INTEGRATION_WORKTREE}}` · started {{DATE}}
Keep this file git-ignored (e.g. `.git/info/exclude`). It is the controller's
recovery map: after compaction or a usage-limit stop, trust it and `git log`.

## Lanes

| Lane | Tasks (in order) | Worktree | Branch | Ports | DB / test DB | Owns |
|------|------------------|----------|--------|-------|--------------|------|
| 0 (hour zero) | {{W01}} | integration | {{INTEGRATION_BRANCH}} | — | — | {{files}} |
| A | {{W02 → W03 → W04}} | {{path}} | {{branch}} | {{8011/5181}} | {{db}} / {{test db}} | {{files}} |

Splits at file boundaries: {{e.g. W05a (component + non-hot callers, lane B) /
W05b (ChargesPanel, LedgerPage — queues behind A's W04 merge)}}

Controller-owned: {{STATE.md, baselines, lockfiles}}
Single-instance resources: {{browser extension → serialized / fallback}}

## Contracts (FROZEN)

- {{name}} — provider {{lane}}, consumers {{lanes}} — `{{file}}`

## Log

One line per event, append-only:

- `{{ID}}: dispatched lane {{X}} (BASE {{sha7}})`
- `{{ID}}: implemented {{sha7}} ({{status}})`
- `{{ID}}: complete ({{base7}}..{{head7}}, verification passed)`
- `{{ID}}: merged into integration ({{merge7}}); forwarded to lanes {{list}}`
