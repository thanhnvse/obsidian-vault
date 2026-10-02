---
tags: [python, concurrency]
status: draft
source: ""
created: 2026-09-28
score: 0.817
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# The GIL lets only one thread run Python bytecode at a time

## Core idea
In CPython the Global Interpreter Lock is held by whichever thread is executing
bytecode. Threads still help while blocked on I/O, because blocking calls release
the lock. CPU-heavy pure-Python code gets no parallel speedup from threads.

## Why choose / why not
- Threads are fine when: the work is mostly waiting (network, disk) and the code base is synchronous.
- Threads are wrong when: the work is CPU-bound pure Python → see [[multiprocessing sidesteps the GIL at the cost of process overhead]].

## Related
- The GIL is why [[asyncio suits I-O-bound work, not CPU-bound work]] competes with threads rather than with processes.
- It exists largely because [[Python memory is managed by reference counting plus a cycle collector]]: refcounts are not thread-safe without a lock.
