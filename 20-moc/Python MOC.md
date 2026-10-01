---
tags: [moc, python]
created: 2026-09-28
---
# Python MOC

## Start here
- [[Python memory is managed by reference counting plus a cycle collector]]

## Concurrency
The one question behind this cluster: *is the work waiting, or computing?*
- [[The GIL lets only one thread run Python bytecode at a time]]: the constraint everything else reacts to
- [[asyncio suits I-O-bound work, not CPU-bound work]]: for waiting
- [[multiprocessing sidesteps the GIL at the cost of process overhead]]: for computing

## Gotchas
- [[Mutable default arguments are evaluated once at definition time]]

## Open questions
- How does free-threaded CPython (PEP 703) change the concurrency cluster? Check against the version actually in use.
