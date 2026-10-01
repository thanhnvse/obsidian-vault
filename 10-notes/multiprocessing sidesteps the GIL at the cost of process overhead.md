---
tags: [python, concurrency]
status: draft
source: ""
created: 2026-09-28
---
# multiprocessing sidesteps the GIL at the cost of process overhead

## Core idea
Each process has its own interpreter and its own GIL, so CPU work runs truly in
parallel. The price: process start-up, higher memory, and every argument and
result must be pickled across the process boundary.

## Why choose / why not
- Choose when: CPU-bound work in chunks big enough that computation dwarfs serialisation cost.
- Don't choose when: tasks are tiny or pass large objects; pickling eats the gain.

## Related
- Exists as the answer to [[The GIL lets only one thread run Python bytecode at a time]].
- The CPU-bound half of the decision whose I/O half is [[asyncio suits I-O-bound work, not CPU-bound work]].
