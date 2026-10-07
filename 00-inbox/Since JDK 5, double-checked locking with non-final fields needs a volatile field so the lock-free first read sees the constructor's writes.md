---
tags: [java, concurrency, memory-model, singleton, interview]
status: draft
author: claude
up: ["[[Concurrency MOC]]", "[[Java core MOC]]"]
source: "https://www.cs.umd.edu/~pugh/java/memoryModel/DoubleCheckedLocking.html"
created: 2026-10-07
review: unjudged
---
# Since JDK 5, double-checked locking with non-final fields needs a volatile field so the lock-free first read sees the constructor's writes

## Core idea
Double-checked locking (DCL) checks the field without the lock, then again inside `synchronized`
before building. The first check is a read that no lock orders, so without `volatile` nothing orders
the constructor's writes before it: a thread that skips the lock can see a non-null reference whose
non-final fields still hold default values, such as `port == 0` instead of 8080. With `volatile` the
chain is complete: the constructor's writes come before the volatile write, the reader's volatile
read sees that write, and its field reads come after. The Pugh et al. declaration says this works
from JDK 5 and not on JDK 4 and earlier. The second check is a separate fix: it stops a waiter from
building a second instance.

## Why choose / why not
- Choose DCL with `volatile` when: construction can fail and must be retried, such as a remote call
  or a file that may appear later; if the constructor throws, the field stays `null` and the next
  call tries again.
- Choose the holder idiom instead when: the instance is lazy and construction cannot fail; the JVM
  does the locking → see [[The holder idiom stays thread-safe without synchronized because the JVM initialises the Holder class once, under its class-initialisation lock]].
- Keep `volatile` even for an immutable object: DCL works without it only when every field is
  `final` and the fast path reads the field once into a local, and that variant is fragile
  (mong manh), because one added non-final field or a second read breaks it silently.
- Don't hand-write it in a Spring application: use a singleton bean injected through the
  constructor → see [[A Spring singleton bean is one instance per container and bean definition, not one per class loader as in the GoF pattern]].

## Interview angle
- Probed as "why does double-checked locking need `volatile`, and why two checks?"
- Common wrong answer: "it is fine without `volatile`, because there is a lock"; the fast path
  reads the field without the lock.
- Strong answer: name the chain (write fields, volatile write, volatile read, read fields), say the
  second check keeps a waiter from building again, and say it is a data race, so a passing test
  proves nothing and jcstress is the tool that collects statistics on it.

## Related
- [[Concurrency MOC]]: DCL without `volatile` is a data race that tests rarely show, so it sits
  with the notes on what happens when threads touch the same data at once.
- [[Java core MOC]]: what the JIT and the CPU may reorder is part of what the JVM does with the code.
- [[volatile makes count++ visible to other threads but does not make it atomic]]: the same keyword
  is enough here, because only the visibility and ordering of one reference matter, and not enough
  there, because `count++` is a read-modify-write.
- [[A thread that sees an object only after its constructor finishes is guaranteed to see the final fields as set, even through a data race]]:
  the declaration's immutable variant works through that guarantee instead of through `volatile`.
- Written up in win-interview: backend/java/docs/java-memory-model.md, sections 2.2 and 2.6.3
