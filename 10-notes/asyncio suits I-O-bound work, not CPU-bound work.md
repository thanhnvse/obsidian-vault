---
tags: [python, concurrency]
status: draft
source: ""
created: 2026-09-28
score: 0.863
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# asyncio suits I/O-bound work, not CPU-bound work

## Core idea
asyncio runs many coroutines on one thread; a coroutine gives up control only at
an `await`. Thousands of concurrent waits are cheap. A CPU-heavy coroutine never
awaits, so it blocks every other coroutine on the loop.

## Why choose / why not
- Choose when: many concurrent network calls, and the libraries in use are async-native.
- Don't choose when: the work is CPU-bound, or key libraries are blocking; one sync call stalls the whole loop.

## Related
- Competes with threads for the same job, because [[The GIL lets only one thread run Python bytecode at a time]] means neither gives CPU parallelism.
- CPU-bound work inside an async app is handed off to [[multiprocessing sidesteps the GIL at the cost of process overhead]].
