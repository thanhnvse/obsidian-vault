---
tags: [java, java-core, collections, interview]
status: draft
author: claude
source: "https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/TreeMap.html"
created: 2026-09-30
score: 0.853
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# TreeMap trades HashMap's constant time for sorted keys and range queries

## Core idea
`TreeMap` is a red-black tree implementation of `NavigableMap`, sorted by the natural ordering of
its keys or by a `Comparator` given at construction. It guarantees log(n) time for
`containsKey`, `get`, `put` and `remove`, whereas `HashMap` offers constant time on average but
no ordering guarantee. In exchange, `NavigableMap` methods such as `floorKey`, `ceilingEntry`
and `subMap` answer nearest-key and range questions that a `HashMap` cannot.

## Why choose / why not
- Choose `TreeMap` when: you need keys in sorted order or a nearest or range lookup, such as
  "the rate that applies on this date" with `floorEntry(date)`, or all events between two
  timestamps with `subMap`.
- Don't choose it when: you only look up exact keys; `HashMap` is faster, and `LinkedHashMap`
  gives a predictable iteration order without sorting.
- Don't give it a `Comparator` that is inconsistent with `equals`, such as one that compares a
  single field: distinct keys that compare as 0 collapse into one entry.

## Interview angle
- Probed as "`HashMap` or `TreeMap`?"; the interviewer wants the ordering versus O(log n) trade,
  then one concrete query that needs `NavigableMap`.
- Common wrong answer: "`TreeMap` is a sorted `HashMap`." It does not hash its keys at all.
- Strong answer: name the red-black tree and the log(n) guarantee, keep `HashMap` as the
  default, and switch only when a query needs order or a range.

## Related
- [[Overriding equals without hashCode makes HashMap lookups miss]]: `HashMap` finds keys with
  `hashCode` and `equals`, while `TreeMap` uses only the comparison, so the two maps break in
  different ways when a key class gets its contracts wrong.
- [[Java collections MOC]]: this is the ordered-map question in the Java core
  collections cluster.
