---
tags: [java, java-core, collections, interview]
status: draft
author: claude
up: ["[[Java collections MOC]]"]
source: "https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/HashMap.html"
created: 2026-10-01
score: 0.84
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# HashMap iteration order follows the buckets and changes when the table resizes

## Core idea
The JDK 21 `HashMap` Javadoc says the class makes no guarantees as to the order of the map, and in
particular does not guarantee that the order will remain constant over time. In OpenJDK 21 a
`HashMap` iterates bucket by bucket, and when the number of entries exceeds capacity times the load
factor the table doubles and each entry either stays at its index or moves to index plus the old
capacity. On OpenJDK 21, the `Integer` keys 3, 17, 2 and 1 iterate as `[17, 1, 2, 3]` in the default
16-bucket table; after nine more keys trigger a resize to 32 buckets, 17 moves behind the others.
Small `Integer` keys only look sorted because `Integer.hashCode()` is the value itself.

## Why choose / why not
- Use `LinkedHashMap` when: output must follow insertion order, such as a JSON response, a CSV export
  or a snapshot test.
- Use `TreeMap`, or sort once at output time, when: output must follow key order.
- Keep `HashMap` when: nothing reads the iteration order; it has the lowest overhead, and a test that
  asserts on its order is asserting on an accident of the table size.

## Interview angle
- Probed as "does `HashMap` keep insertion order?", or via a test that passes on small data and fails
  on production-sized data.
- Common wrong answer: "it keeps insertion order", or "it is sorted for small integer keys."
- Strong answer: it iterates in bucket order, which depends on the hashes and the table size, so a
  resize reorders it; the Javadoc promises no stable order, so use `LinkedHashMap` or `TreeMap` when
  order matters.

## Related
- [[LinkedHashMap in access order with removeEldestEntry is an LRU cache]]: the same class in its
  default insertion-order mode is the fix when callers need a predictable order.
- [[TreeMap trades HashMap's constant time for sorted keys and range queries]]: the choice when the
  required order is key order, at the cost of O(log n) lookups.
