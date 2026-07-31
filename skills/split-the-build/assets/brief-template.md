# Brief: {{PERSON_NAME}} — {{SLICE_NAME}}

> Branch: `{{BRANCH_NAME}}` · Estimated: {{HOURS}}h of {{TIME_REMAINING}} ·
> Strengths matched: {{STRENGTHS}}

## Mission

{{One outcome-phrased sentence: what exists and works when you're done.}}

## Files you own

You are the only person editing these. Conflicts here are your fault; edits
elsewhere are worse.

```
{{path/one}}
{{path/two/}}          # whole directory
{{path/new-file}}      # you create this
```

## Files you must NOT touch

| Path | Owner | If you need a change |
|------|-------|----------------------|
| {{path}} | {{person}} | Ask them / follow contract change protocol |

## Contracts you PROVIDE

Others code against these today. Breaking them breaks teammates — changes
only via the change protocol.

```{{lang}}
{{Paste full frozen contract definition(s) here.}}
```

## Contracts you CONSUME

Code against these signatures. Stubs behind them already work on main.

```{{lang}}
{{Paste full frozen contract definition(s) here.}}
```

## Hour-zero stubs you can rely on

- {{stub — what it fakes, where it lives}}

## Definition of done

Runnable without anyone else's slice finished:

- [ ] `{{command}}` → {{expected output}}
- [ ] {{manual check: open X, see Y}}

## Demo responsibility

At the demo, your slice delivers: {{the sentence(s) of the demo script this
slice owns}}.

## If behind, cut in this order

1. {{first thing to drop}}
2. {{second}}
   — never cut: {{the part feeding the one demoable thing}}

## If you're blocked

1. **Blocked on another slice's unfinished work?** Don't wait — code against
   the frozen contract and the hour-zero stub. Integration freeze is when
   real implementations replace stubs, not before.
2. **Stub missing or broken?** That's a hair-on-fire bug for the stub's
   owner. Ping them immediately; meanwhile hardcode a fake in *your own
   files* and tag it `// TEMP-UNBLOCK` so it's greppable at merge time.
3. **Contract doesn't cover your case?** Do not silently extend it. Ping
   provider + consumers; follow the change protocol. While waiting, build
   the parts that don't depend on the gap.
4. **Timebox: 30 minutes.** Blocked longer with no answer → skip to your
   next independent task and post what you skipped.
5. **Never fix a blocker by editing files you don't own.**

## Merge window

Merge order position: {{n}} of {{N}} — after {{person}}, before {{person}}.
Integration freeze: {{time}}. After freeze: integration fixes only.
