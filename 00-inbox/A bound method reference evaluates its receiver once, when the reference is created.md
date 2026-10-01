---
tags: [java, java-core, lambda, interview]
status: draft
author: claude
up: ["[[Java core MOC]]"]
source: "https://docs.oracle.com/javase/specs/jls/se21/html/jls-15.html#jls-15.13.3"
created: 2026-10-01
score: 0.915
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A bound method reference evaluates its receiver once, when the reference is created

## Core idea
JLS §15.13.3 says that when a method reference expression begins with an expression name or a
primary, such as `this.handler::handle`, that subexpression is evaluated first, when the method
reference expression itself is evaluated. If the receiver evaluates to `null`, a
`NullPointerException` is raised at that point, before the method is ever invoked. Each later call
uses the target reference determined at creation time. A lambda such as `x -> this.handler.handle(x)`
instead reads `this.handler` every time it is called. So after `this.handler` is reassigned, the
method reference keeps calling the old handler while the lambda calls the new one, and a `null`
receiver makes the method reference fail where it is created but the lambda fail only when it is called.

## Why choose / why not
- Choose the bound method reference when: the receiver is fixed for the reference's lifetime, such as
  a final field or a local; it is shorter and fails fast on a `null` receiver.
- Choose a lambda when: the receiver can change, such as a handler swapped by reconfiguration or a
  field that is set later; the lambda reads the current value on each call.
- Don't convert `x -> field.method(x)` to `field::method` mechanically when: `field` is mutable or may
  still be `null` where the reference is created; the conversion changes behaviour without a
  compiler warning.

## Interview angle
- Probed as "are `x -> handler.handle(x)` and `handler::handle` the same?", often with a stack trace
  that points at the line creating the reference rather than the call.
- Common wrong answer: "a method reference is just shorthand for the equivalent lambda."
- Strong answer: a bound reference evaluates and captures its receiver once, at creation, so it can
  throw `NullPointerException` early and keeps calling a stale receiver; the lambda re-reads it per call.

## Related
- [[A method reference bound to this is a new, unequal object each time it is evaluated]]: that note
  is about the identity of the reference object; this one is about when its receiver is evaluated,
  and both come from the same JLS §15.13.3 evaluation rule.
- [[Mutable default arguments are evaluated once at definition time]]: Python has the same
  evaluated-once-at-creation trap for default arguments, so the bug shape carries across languages.
