---
tags: [java, java-core, collections, interview]
status: draft
author: claude
up: ["[[Java collections MOC]]"]
source: "https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ArrayList.html"
created: 2026-10-01
score: 0.85
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Removing the second-to-last element inside a for-each loop skips the last element without a ConcurrentModificationException

## Core idea
The JDK 21 `ArrayList` Javadoc says its iterators are fail-fast but throw
`ConcurrentModificationException` only on a best-effort basis, so a program must not depend on that
exception for its correctness. In the OpenJDK 21 source, the iterator's `hasNext()` only compares
its cursor with the list's current size, and the `modCount` check that throws runs in `next()`.
When a for-each loop calls `list.remove(x)` on the second-to-last element, the size drops to the
cursor value, so `hasNext()` returns false and the loop ends without calling `next()` again. The
last element is never visited and no exception is thrown; on OpenJDK 21 `LinkedList` behaves the
same way.

## Why choose / why not
- Use `list.removeIf(predicate)` when: you remove elements by a condition; it removes them in one
  pass with no iterator in your code.
- Use an explicit `Iterator` and its `remove()` when: the removal decision needs more than a
  predicate, such as a side effect per removed element; the iterator then stays in step with the list.
- Don't treat a passing test as proof when: the loop calls `list.remove` inside a for-each; whether
  it throws depends on which element is removed, so a test that removes a middle element throws while
  production data that hits the second-to-last element silently skips one.

## Interview angle
- Probed as "what happens if you remove an element inside a for-each loop?"
- Common wrong answer: "it always throws `ConcurrentModificationException`, so the bug cannot go
  unnoticed."
- Strong answer: usually it throws, but fail-fast is best-effort by contract; removing the
  second-to-last element ends the loop early and skips the last one. Use `removeIf` or
  `Iterator.remove()`.

## Related
- [[ArrayList is the default List because LinkedList walks its nodes on every indexed access]]: the
  two lists differ in layout, but their iterators share this cursor-versus-size check, so switching
  list type does not fix the skipped element.
- [[LinkedHashMap in access order with removeEldestEntry is an LRU cache]]: there even a `get` is a
  structural modification, which is the same kind of change that a fail-fast iterator reports.
