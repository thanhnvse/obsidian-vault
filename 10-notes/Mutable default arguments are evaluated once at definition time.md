---
tags: [python, gotcha]
status: draft
source: ""
created: 2026-09-28
score: 0.787
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Mutable default arguments are evaluated once at definition time

## Core idea
`def f(x, acc=[])` creates the list once, when `def` runs. Every call that omits
`acc` shares the same list, so it grows across calls.

```python
def f(x, acc=None):
    if acc is None:
        acc = []
    acc.append(x)
    return acc
```

## Why choose / why not
- Use `None` as the sentinel whenever the default is mutable.
- Deliberately sharing the default (as a cache) works but surprises readers; prefer an explicit cache.

## Related
- Same root cause as other shared-reference surprises: names bind to objects, see [[Python memory is managed by reference counting plus a cycle collector]].
