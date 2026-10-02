---
tags: [java, java-core, lambdas, interview]
status: draft
author: claude
up: ["[[Java core MOC]]"]
source: "https://docs.oracle.com/javase/specs/jls/se21/html/jls-15.html#jls-15.13.3"
created: 2026-09-30
score: 0.867
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A method reference bound to this is a new, unequal object each time it is evaluated

## Core idea
A bound method reference like `this::onEvent` captures its receiver, here `this`. JLS §15.13.3
evaluates that receiver when the method reference expression is evaluated, and allows either a new
object or an existing one as the result. On OpenJDK 21 a capturing method reference produces a new
instance of its hidden class on every evaluation, and that class does not override `equals`, so two
evaluations of `this::onEvent` are neither the same object nor equal. Code that calls
`bus.register(this::onEvent)` and later `bus.unregister(this::onEvent)` therefore removes nothing:
the listener stays registered and keeps `this` reachable.

## Why choose / why not
- Keep the reference in a field when: you must unregister it later, for example
  `private final Runnable listener = this::onEvent;`, and pass that same object to both calls.
- Prefer an API that returns a handle when: you can choose one; closing a returned subscription
  does not depend on the identity of a lambda.
- Don't use a lambda or method reference as a map key or set element when: lookups must match
  later evaluations; whether two evaluations share an object is an implementation choice.

## Interview angle
- Probed as a debugging question: "why does this listener keep firing after we unregister it?" or
  "why does this listener registry keep growing?".
- Common wrong answer: "method references are singletons, so the second one is the same object."
- Strong answer: a bound reference captures `this`, so each evaluation creates a new object without
  value equality; keep the reference, and name the leak it causes.

## Related
- [[A lambda is linked through invokedynamic at run time, not compiled to its own class file]]:
  that call site is what returns a fresh object per evaluation once the lambda captures something.
- [[A Java memory leak is an object that stays reachable after the program stops needing it]]: a
  listener that is never really removed is one of those leaks, because it keeps `this` reachable.
- [[Inside a lambda, this refers to the enclosing instance, not to the lambda]]: `this::onEvent`
  captures the same enclosing instance that note describes.
