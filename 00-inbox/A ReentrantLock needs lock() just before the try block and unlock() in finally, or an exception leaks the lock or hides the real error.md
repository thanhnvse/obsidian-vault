---
tags: [java, concurrency, locks, reentrantlock, interview]
status: draft
author: claude
up: ["[[Concurrency MOC]]"]
source: "https://docs.oracle.com/en/java/javase/23/docs/api/java.base/java/util/concurrent/locks/Lock.html"
created: 2026-10-07
review: unjudged
---
# A ReentrantLock needs lock() just before the try block and unlock() in finally, or an exception leaks the lock or hides the real error

## Core idea
The JVM releases a `synchronized` monitor on every exit, but a `Lock` is released only by your
`unlock()`. The `Lock` javadoc idiom is `lock()` as the last statement before the `try` and
`unlock()` as the first statement in the `finally`, and the lab breaks each half. With `unlock()`
outside `finally`, an exception leaves the lock held even after its owner thread has died, and every
later thread waits forever with nothing reported; a pool thread that survives the exception still
owns it. With `lock()` inside the `try`, a failed acquisition, such as `lockInterruptibly()` on an
interrupted thread, makes the `finally` unlock a lock that is not held, and the resulting
`IllegalMonitorStateException` replaces the real exception, which is neither its cause nor suppressed.

## Why choose / why not
- Choose `synchronized` when: you need only mutual exclusion over one block; it releases on every
  exit, so it cannot leak a lock, and it is shorter. It is the usual choice.
- Choose `ReentrantLock` when: you need a timed `tryLock`, `lockInterruptibly`, a fair queue,
  several conditions, a lock released in another method, or on JDK 21 to 23 a lock held across
  blocking I/O in virtual threads; the price is this discipline.
- With `tryLock`, test the result first and put the `try` inside the `if`, so the code never
  unlocks a lock it did not get.

## Interview angle
- Probed as "why must `unlock()` be in `finally`, and why is `lock()` outside the `try`?"
- Weak answer: stop at "unlock in `finally`" and put `lock()` inside the `try`.
- Strong answer: give both halves with their symptoms, a leaked lock that outlives the thread and a
  hidden cause, plus the `tryLock` shape; in a dump the waiters are `WAITING (parking)`, and when the
  owner is still alive, such as a pool thread, the leaked lock shows under its "Locked ownable
  synchronizers", which needs `jcmd <pid> Thread.print -l`; if the owner died, no live thread lists it.

## Related
- [[Concurrency MOC]]: a leaked lock is a hang with no error message, the thread-level cousin of the
  data-locking problems in that map.
- [[try-with-resources keeps the body's exception and suppresses close failures, while a throwing finally replaces it]]:
  the same rule hides the real error here, because a throwing `finally` replaces the exception in
  flight and the `IllegalMonitorStateException` replaces the `InterruptedException`.
- [[On JDK 21 to 23 a virtual thread that blocks inside synchronized stays pinned to its carrier thread, and JDK 24 removes that]]:
  that pinning is the reason to pay for this idiom with a `ReentrantLock` on JDK 21 to 23.
- [[A wait() or await() call belongs in a while loop, because a woken thread can find its condition false again]]:
  a `Condition` belongs to this lock and `await()` returns holding it again.
- Written up in win-interview: backend/java/docs/locks-and-thread-hazards.md, sections 2.3 and 4
