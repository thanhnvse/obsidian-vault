---
tags: [java, java-core, lambda, interview]
status: draft
author: claude
source: "https://docs.oracle.com/javase/specs/jls/se21/html/jls-15.html#jls-15.27.2"
created: 2026-09-30
score: 0.77
review: "borderline"
score_reasons: ["links_reasoned: 0.56 (borderline)", "numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Inside a lambda, this refers to the enclosing instance, not to the lambda

## Core idea
A lambda body is lexically scoped: JLS §15.27.2 says that, unlike code in an anonymous class,
names and the `this` and `super` keywords in a lambda body mean the same as in the surrounding
context. So `this`, or an unqualified `toString()`, inside a lambda written in an instance method
refers to the enclosing object. Inside an anonymous class body, `this` is the anonymous class
instance, and the enclosing object is reached with a qualified `Outer.this`.

## Why choose / why not
- Choose a lambda when: the callback works with the enclosing object's fields and methods; they
  are in scope exactly as in the surrounding method, with no `Outer.this`.
- Choose an anonymous class when: the callback must refer to itself, for example a listener that
  deregisters itself with `removeListener(this)`, or a `TimerTask` that calls its own `cancel()`.
- Don't convert an anonymous class to a lambda mechanically when its body uses `this`,
  `toString()` or `hashCode()` unqualified: after the change those calls go to the enclosing
  object, and the code still compiles.

## Interview angle
- Probed as "what does this print?", with `this` printed from a lambda and from an anonymous class
  inside the same instance method.
- Common wrong answer: "`this` in a lambda refers to the lambda object."
- Strong answer: "a lambda is lexically scoped; an anonymous class opens a new class scope", then
  derive both `this` results and the need for `Outer.this` from that one sentence.

## Related
- [[A lambda is linked through invokedynamic at run time, not compiled to its own class file]]:
  that note is the bytecode difference between a lambda and an anonymous class; this note is the
  scoping difference, and interviewers usually ask for both.
- [[Java core MOC]]: this belongs to the lambda questions in the Java core section.
