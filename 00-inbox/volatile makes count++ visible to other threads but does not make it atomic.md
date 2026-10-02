---
tags: [java, concurrency, race-condition, interview]
status: draft
author: claude
up: ["[[Concurrency MOC]]"]
source: "https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/atomic/AtomicInteger.html"
created: 2026-10-01
score: 0.893
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# volatile makes count++ visible to other threads but does not make it atomic

## Core idea
`count++` on a field is three steps: read the value, add one, and write the result back. Declaring
the field `volatile` makes each read see the latest write to that field by any thread, and makes
each single read and each single write atomic, but it does not join the three steps into one.
Another thread can still write between this thread's read and its write, so two threads that each
increment once can leave the count at 1: `volatile` gives visibility, not atomicity of a
read-modify-write. Only an operation that performs the whole read-modify-write atomically, such as
`AtomicInteger.incrementAndGet()`, or a lock around the three steps closes that window.

## Why choose / why not
- Use `volatile` when: one thread writes and other threads only read, such as a shutdown flag or a
  reference to a freshly published configuration; visibility is all that is needed.
- Use `AtomicInteger` or `AtomicLong` when: several threads update one value; use `LongAdder` for a
  hot counter that many threads increment and few read.
- Use a lock when: the invariant spans several fields, which no single atomic variable can cover.
- Don't use any of them when: the shared state lives in a database used by several instances; each
  JVM has its own memory, so the guard belongs in the database.

## Interview angle
- Probed as "why is `count++` not thread-safe, and does `volatile` fix it?"
- Common wrong answer: "`volatile` makes it thread-safe."
- Strong answer: read, add, write leaves a window; `volatile` gives visibility, not atomicity;
  `AtomicInteger` closes the window with compare-and-set (detect and retry) and `synchronized` with
  mutual exclusion (wait); both stop at the JVM boundary.

## Related
- [[A synchronized block cannot prevent a lost update between two application instances]]: it is
  related because it shows the same lost increment between two JVMs, where `volatile`,
  `AtomicInteger` and `synchronized` all stop working, since each lives in one JVM's memory.
- [[A version column detects a lost update at write time instead of blocking the other writer]]: it
  is related because `AtomicInteger.compareAndSet` applies the same rule in memory that a version
  column applies in the database: write only if the value is still the one that was read, and
  otherwise reread and retry.
