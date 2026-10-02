---
tags: [java, java-core, lambda, interview]
status: draft
author: claude
up: ["[[Java core MOC]]"]
source: "https://docs.oracle.com/javase/specs/jls/se21/html/jls-15.html#jls-15.27.2"
created: 2026-10-01
score: 0.881
review: "borderline"
score_reasons: ["atomic: 0.59 (borderline)", "numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 2
---
# A lambda captures the value of a local variable, which is why the variable must be effectively final

## Core idea
JLS §15.27.2 requires every local variable, formal parameter or exception parameter that a lambda
uses but does not declare to be final or effectively final. The JLS explains that the restriction
prohibits access to dynamically-changing local variables, whose capture would likely introduce
concurrency problems. On OpenJDK 21 the captured value is copied into a field of the object that
implements the lambda, so the lambda can still run after the method that declared the variable has
returned, or on another thread. What is copied is the variable's value, so the rule covers the
variable and not the object it refers to: a captured `List` variable cannot be reassigned, but the
lambda can still add elements to the list.

## Why choose / why not
- Capture a local when: the lambda only reads a value fixed before it was created, such as a
  request id or a threshold; that is the case the rule is designed for.
- Use `sum`, `reduce` or `collect` instead of a captured accumulator when: you are tempted to work
  around the rule with a one-element array or an `AtomicInteger`; the stream combines partial results
  without shared mutable state.
- Don't mutate a captured collection when: the lambda runs in a parallel stream, an executor or a
  `CompletableFuture`; the variable is effectively final, but the `ArrayList` behind it is not
  thread-safe, so writes from several threads race.

## Interview angle
- Probed as "why must variables used in a lambda be effectively final?"
- Common wrong answer: "lambdas capture variables by reference", or "it is a limitation of the
  compiler with no reason behind it."
- Strong answer: the lambda gets a copy of the value and may run later or on another thread; if the
  variable could still change, the copy and the variable would disagree. The object behind a captured
  reference can still change, which is where the data races come from.

## Related
- [[A lambda is linked through invokedynamic at run time, not compiled to its own class file]]: that
  note explains how the lambda object is created at run time; this note explains what the object
  holds once it exists, a copy of each captured local.
- [[Parallel streams share the JVM-wide common ForkJoinPool]]: that note explains why the lambdas of a
  parallel stream run on several threads at once, which is exactly when adding to a captured list
  becomes a data race.
