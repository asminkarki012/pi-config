---
name: state-first-debugging
description: Forces whole-system thinking before coding and stops repeated debugging loops. Use when a feature spans two or more storages (database, cache, external service, agent memory), when a bug survives a second attempt, when the same question is asked twice, or when someone says "it works now" without having seen the stored data.
---

# State-First Debugging

Two failure modes cost the most time in real sessions, and both are cheap to prevent:

1. **Designing before mapping the system.** A feature that touches two documents which can be created in either order has ordering bugs by construction, not by accident.
2. **Patching before looking.** Each code change made from a guess costs a test cycle; a dump of the stored state answers the same question in one command.

This skill is a set of stop conditions and artifacts, not advice.

## Rule 1 — Draw the state map before writing code

Produce this, maximum ten lines, before the first edit:

```
entity / storage | source of truth? | written by | read by | sync or async
```

Plus the arrows between them, e.g. `browser → route → tool → db → external service → its own table`.

If two or more entities can be created in either order, that is the feature's main design problem — solve it before any other work.

Do not proceed while any row is unknown. "I will find out as I go" is the exact condition that produces three rewrites.

## Rule 2 — Write the ordering matrix, build the worst case first

List every creation order a user can produce:

| # | Order | Result |
|---|---|---|
| 1 | A then B | |
| 2 | B then A | |
| 3 | A and B in one action | |
| 4 | B deleted and recreated | |

Build **#2 or #4 first**. Order #1 will work anyway; it is the one that hides bugs.

Any value that must exist before its owner exists gets one of exactly three treatments. Choose explicitly:

1. carry it with the nearest long-lived container (conversation, session, parent record)
2. park it and flush when the owner appears
3. recompute it from a durable source when needed

## Rule 3 — Stop conditions

Stop editing code and switch to inspection when any of these is true:

- three attempts at the same bug
- the same question returns twice
- a field that "should be set" is empty
- a value is correct in one place and wrong in another

The stop is not optional. Continuing past it is how a one-line fix becomes an afternoon.

## Rule 4 — Dump before patch

One command that prints every layer, values only, no prose:

```
thread   flags + pending state
owner    id, link, switch
children counts and distribution
external enforce flag, row count
```

Read the dump out loud as a sentence. The broken arrow is usually visible in it. Only then touch code.

No dump, no edit. Console logging is not a dump: it shows the path the code took, not the state that was stored.

## Rule 5 — Acceptance is a number or an exact string

"Tuition works" is not acceptance. This is:

```
student 303474019 → the Hebrew refusal message
document → 65 entries, 64 paid, 1 unpaid
external table → 65 rows, enforce = 1
```

Anything that cannot be written this way is a product decision, not a test. Surface it as a question instead of implementing a guess.

## Rule 6 — Verify the layer under test, not the UI

If the logic lives in a function, call the function. Draft approvals, form submissions and page reloads are minutes each and verify the UI, not the rule.

Prefer: a script that calls the service, prints the result, and runs in seconds.

## Rule 7 — Every silent write needs a read-back or a report

Best-effort writes that log and continue produce bugs that look like "the card is broken". When a later step depends on a write, verify it and return an explicit failure instead of returning success.

## Rule 8 — Close with three lines

At the end of the feature, write:

1. what surprised me
2. what I assumed without checking
3. what I will dump first next time

These three lines are the only reliable way this skill gets faster over time.

## Anti-patterns, with the correct replacement

| Anti-pattern | Do instead |
|---|---|
| adding another hook/trigger to fix an ordering bug | decide which side owns the value (Rule 2) |
| debugging with console.log | dump the stored state (Rule 4) |
| verifying through the UI | call the function directly (Rule 6) |
| "it works now" | show the dump that proves it (Rule 4) |
| designing while coding | state map first (Rule 1) |
| fixing the third symptom before mapping | stop condition (Rule 3) |
| one global flag then per-record then metadata | pick the owner before writing any of them (Rule 1) |

## Session checklist

Before code:

- [ ] state map, ten lines, every row known
- [ ] ordering matrix, worst case identified
- [ ] source of truth named for each value
- [ ] acceptance written as numbers or strings

During work:

- [ ] three failed attempts → stop, dump
- [ ] same question twice → dump, do not re-ask
- [ ] new storage introduced → add it to the map

At close:

- [ ] dump proves the acceptance criteria
- [ ] three-line close-out written
- [ ] map updated where the code diverged from it
