---
tags: [java, java-core, streams, interview]
status: draft
author: claude
up: ["[[Java core MOC]]"]
source: "https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Stream.html#reduce(T,java.util.function.BinaryOperator)"
created: 2026-10-01
score: 0.87
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A reduce seed that is not a true identity is added once per chunk in a parallel stream

## Core idea
The JDK 21 Javadoc of `Stream.reduce(identity, accumulator)` requires the identity value to be an
identity for the accumulator: for every `t`, `accumulator.apply(identity, t)` must equal `t`. The
Javadoc describes the reduction as a loop that starts from the identity, but says it is not
constrained to execute sequentially. In OpenJDK 21 a parallel stream splits its source into chunks,
reduces each chunk starting from the identity, and then combines the partial results, so
`reduce(10, Integer::sum)` adds 10 once on a sequential stream but once per chunk on a
parallel stream. The same code then returns a larger total after someone adds `parallel()`.

## Why choose / why not
- Pass a true identity, such as `0` for addition, `1` for multiplication or `""` for concatenation,
  when: you call `reduce(identity, op)`; the result then does not depend on how the stream is split.
- Add the starting value outside the reduction, as in `10 + stream.reduce(0, Integer::sum)`, when:
  the business rule needs an offset or an opening balance.
- Use `sum()` on an `IntStream` or a collector when: the reduction is a plain total; there is no seed
  to get wrong and no boxing.

## Interview angle
- Probed as "this sum is correct sequentially and wrong in parallel; why?", or as "what must the first
  argument of `reduce` be?"
- Common wrong answer: "the identity is just the starting value of the loop."
- Strong answer: it must be an identity for the operation, because a parallel stream starts each
  chunk from it; a non-identity seed is applied once per chunk, and the operation must also be
  associative.

## Related
- [[Parallel streams share the JVM-wide common ForkJoinPool]]: the chunks are the fork/join tasks that
  pool runs, so the number of times the seed is applied depends on how the source was split.
- [[Stream intermediate operations are lazy and run only when a terminal operation starts]]: `reduce`
  is the terminal operation that starts the split and the per-chunk reductions.
