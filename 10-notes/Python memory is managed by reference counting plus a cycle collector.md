---
tags: [python, memory]
status: draft
source: ""
created: 2026-09-28
---
# Python memory is managed by reference counting plus a cycle collector

## Core idea
Every CPython object carries a reference count; it is freed the moment the count
hits zero. Reference cycles never reach zero, so a separate generational
collector finds and frees them periodically.

## Why choose / why not
- Upside: deterministic, immediate cleanup in the common case.
- Downside: a count update on every assignment, and cycles depend on the collector.

## Related
- Non-thread-safe refcounts are a core reason [[The GIL lets only one thread run Python bytecode at a time]] exists.
