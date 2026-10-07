---
tags: [java, concurrency, memory-model, immutability, interview]
status: draft
author: claude
up: ["[[Java core MOC]]", "[[Concurrency MOC]]"]
source: "https://docs.oracle.com/javase/specs/jls/se21/html/jls-17.html#jls-17.5"
created: 2026-10-07
review: unjudged
---
# A thread that sees an object only after its constructor finishes is guaranteed to see the final fields as set, even through a data race

## Core idea
JLS 17.5 says that a thread that can only see a reference to an object after the object is
completely initialised, meaning its constructor has finished, is guaranteed to see the correctly set
values of its `final` fields, and versions of what they refer to that are at least as up to date as
those fields, even when the reference reached it through a data race such as a plain static field.
In the JLS example, `final int x` and plain `int y` are set in the same constructor: a reader of the object sees `x` correctly but may not see `y`. This is
why immutable objects such as `String` and records can be shared without a lock. It is a separate
guarantee, not a happens-before edge, and it starts only when the constructor ends: in the lab, a
thread handed `this` before the constructor finished reads 0 from a `final int port`, deterministically.

## Why choose / why not
- Make fields `final` when: the object is shared between threads and its state must not change;
  readers see the constructor's values even if the reference is published through a data race,
  provided `this` did not escape the constructor.
- Don't let `this` escape the constructor when you rely on it: registering a listener, starting a
  thread or putting `this` in a shared map breaks the guarantee; do that in a factory method after
  construction.
- Don't read `final` as covering later changes: a `final` reference to a mutable object guarantees
  what the constructor wrote, and changes made after publication need their own edge, such as a
  lock, a `volatile` field or a concurrent collection.

## Interview angle
- Probed as "what is safe publication?" or "why are immutable objects thread-safe?"
- Common wrong answers: "`final` fields are always seen correctly" (only if `this` did not escape
  the constructor) and "`final` creates a happens-before edge" (it does not; JLS 17.5.1 uses a
  special ordering that does not chain with other edges).
- Strong answer: state the rule with its condition, say it holds even through a data race, give the
  escaping-`this` counter-example, and name the other safe-publication idioms: a static
  initialiser, a `volatile` or atomic reference, a lock, a concurrent collection.

## Related
- [[Java core MOC]]: how the JVM makes a constructor's writes visible is part of what it does with
  the code, and immutable classes depend on it.
- [[Concurrency MOC]]: safe publication is the thread-level question of how a shared object gets to
  another thread intact.
- [[String immutability comes from a private final byte array that no method changes or exposes, not from the class being final]]:
  it relies on this guarantee to let a `String` cross threads without locks; this note states the
  rule and the condition that `this` must not escape.
- [[Since JDK 5, double-checked locking with non-final fields needs a volatile field so the lock-free first read sees the constructor's writes]]:
  the declaration's immutable variant of DCL, with only `final` fields, works through this guarantee
  and not through `volatile`.
- Written up in win-interview: backend/java/docs/java-memory-model.md, sections 2.4 and 2.5
