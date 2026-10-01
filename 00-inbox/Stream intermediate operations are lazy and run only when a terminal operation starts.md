---
tags: [java, java-core, streams, interview]
status: draft
author: claude
source: "https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/package-summary.html"
created: 2026-09-30
score: 0.863
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 2
---
# Stream intermediate operations are lazy and run only when a terminal operation starts

## Core idea
Intermediate operations such as `filter` and `map` are always lazy: each one returns a new stream
and does no work, and traversal of the source does not begin until a terminal operation such as
`collect` or `findFirst` runs. Because nothing runs early, the steps of a filter-map-sum pipeline
are fused into a single pass over the data instead of one pass per operation. Laziness also lets
a short-circuiting operation such as `findFirst` or `limit` stop after examining just enough
elements, so a pipeline over the infinite `Stream.iterate` source can finish. Stateful operations
such as `sorted` are the exception, because they may need the entire input before producing a
result.

## Why choose / why not
- Rely on laziness when: a search can end early, such as `filter(...).findFirst()` over a large
  list; only the elements up to the first match are examined.
- Don't expect an early exit when: a stateful `sorted` sits before `findFirst`; it consumes the
  whole input first, so filter before sorting, or use `min` with a comparator instead.
- Don't put side effects that must happen into `peek` or `map`: a lazy step runs only for the
  elements the terminal operation pulls, and since Java 9 `count` may skip the pipeline entirely.

## Interview angle
- Probed as "what does `list.stream().peek(System.out::println).count()` print?"; from Java 9 on
  the answer may be nothing, because `count` can be taken from the list size.
- Common wrong answer: "each intermediate operation makes its own pass over the collection."
- Strong answer: name the three effects of laziness (fusion into one pass, short-circuiting, and
  skipped work) and the stateful exception.

## Related
- [[Java core MOC]]: laziness is the first stream question in the Java core section,
  and it underlies every later question about stream performance.
