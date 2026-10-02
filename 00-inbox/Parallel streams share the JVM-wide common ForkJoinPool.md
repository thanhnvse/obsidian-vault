---
tags: [java, java-core, streams, interview]
status: draft
author: claude
source: "https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ForkJoinPool.html"
created: 2026-09-30
score: 0.867
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Parallel streams share the JVM-wide common ForkJoinPool

## Core idea
A parallel stream splits its source and runs the pieces as fork/join tasks; the Java Tutorial
states that methods in the `java.util.stream` package use the fork/join framework. A task forked
outside any `ForkJoinPool` runs in `ForkJoinPool.commonPool()`, the pool used by every
`ForkJoinTask` not explicitly submitted to a specified pool, so all parallel streams in one JVM
share the same worker threads, and the thread that started the terminal operation also takes part.
The `CompletableFuture` async methods without an explicit `Executor` run in that same common pool.
Its parallelism is set once per JVM with the system property
`java.util.concurrent.ForkJoinPool.common.parallelism`.

## Why choose / why not
- Choose a parallel stream when: the work is CPU-bound, the data set is large, and a measurement
  on the target machine shows a real speed-up over the sequential stream.
- Don't choose it when: the lambdas block on a database or HTTP call; the `ForkJoinTask` Javadoc
  says such tasks should not perform blocking I/O, and a blocked worker is unavailable to every other
  parallel stream and `CompletableFuture` in the JVM. Use an `ExecutorService` sized for the I/O.
- Don't choose it when: the code runs on the request threads of a busy server; concurrent requests
  all compete for the one common pool, so the speed-up per request shrinks as load rises.

## Interview angle
- Probed as "would you call `parallelStream()` inside a request handler?"; the interviewer wants
  the shared common pool first, then the blocking I/O risk.
- Common wrong answer: "each parallel stream gets its own threads."
- Strong answer: name the common pool, its sharing with `CompletableFuture` async methods that take
  no executor, and the parallelism property, then say you would measure before switching.

## Related
- [[Stream intermediate operations are lazy and run only when a terminal operation starts]]: a
  parallel pipeline is just as lazy, so the split into fork/join tasks happens only when the
  terminal operation starts.
- [[Java core MOC]]: this is the stream performance follow-up in the Java core
  section.
