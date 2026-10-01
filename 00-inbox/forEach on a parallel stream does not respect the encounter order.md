---
tags: [java, java-core, streams, interview]
status: draft
author: claude
up: ["[[Java core MOC]]"]
source: "https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Stream.html#forEach(java.util.function.Consumer)"
created: 2026-10-01
score: 0.895
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 2
---
# forEach on a parallel stream does not respect the encounter order

## Core idea
The JDK 21 Javadoc of `Stream.forEach` says its behavior is explicitly nondeterministic: for parallel
stream pipelines it does not guarantee to respect the encounter order of the stream, as doing so
would sacrifice the benefit of parallelism. The action may run for any element at whatever time and
in whatever thread the library chooses, and if it touches shared state it must provide its own
synchronization. So `list.parallelStream().forEach(action)` may apply the action to the elements in
any order, and the order can differ from one run to the next. The ordered counterpart is
`forEachOrdered`, which processes the elements one at a time in encounter order when the stream has
one.

## Why choose / why not
- Use `forEach` on a parallel stream when: the order does not matter and the action is independent
  per element, such as sending each element to a thread-safe sink.
- Use `forEachOrdered` when: an ordered side effect is required, such as writing lines of a report;
  it keeps the order but gives up most of the parallel speed-up for that step.
- Collect instead of adding from `forEach` when: the goal is a list or a map; `toList()` or `collect`
  keeps the order and avoids the race of many threads writing into one `ArrayList`.

## Interview angle
- Probed as "what does `list.parallelStream().forEach(System.out::println)` print?"
- Common wrong answer: "the elements in list order, just faster."
- Strong answer: some order that can differ between runs, because `forEach` is explicitly
  nondeterministic in parallel; use `forEachOrdered` or collect, and never mutate a shared collection
  from `forEach`.

## Related
- [[Parallel streams share the JVM-wide common ForkJoinPool]]: that note explains that a parallel
  stream's work runs on the common pool's workers and the caller thread at the same time, which is
  why `forEach` cannot promise an order.
- [[A reduce seed that is not a true identity is added once per chunk in a parallel stream]]: that
  note is a second way the same pipeline changes its result once `parallel()` is added, there through
  the seed and here through the order of side effects.
