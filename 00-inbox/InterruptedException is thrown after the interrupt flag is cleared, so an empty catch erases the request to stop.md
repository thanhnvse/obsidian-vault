---
tags: [java, java-core, exceptions, threads, interview]
status: draft
author: claude
up: ["[[Java core MOC]]", "[[Concurrency MOC]]"]
source: ""
created: 2026-10-07
review: unjudged
---
# InterruptedException is thrown after the interrupt flag is cleared, so an empty catch erases the request to stop

## Core idea
`InterruptedException` is a checked exception that a blocking call such as `CountDownLatch.await`
throws after the thread's interrupt flag has been reset, so the exception is the only record that someone asked the thread to stop. A `catch` that
does nothing erases that record: the next blocking call does not see an interrupt and the thread
carries on. The lab proves both halves: once the exception is thrown the flag is clear, and calling
`Thread.currentThread().interrupt()` in the `catch` lets the next blocking call see the interrupt.
The fix is to restore the flag or to rethrow.

## Why choose / why not
- Rethrow when: the method can declare `InterruptedException`; the caller then decides what
  stopping means.
- Restore the flag with `Thread.currentThread().interrupt()` when: the method cannot declare the
  exception, for example inside a lambda for a standard functional interface, which cannot throw a
  checked exception.
- Don't leave the `catch` empty: the symptom is a shutdown that hangs, because a pool that wants to
  stop waits for a task that never learns it was asked to.

## Interview angle
- Asked as "what does `Thread.interrupt` have to do with exceptions?".
- Common wrong answer: catch it, do nothing and carry on, as if catching the exception handled it.
- Strong answer: the exception arrives after the flag was cleared, so the `catch` must restore the
  flag or rethrow; name the symptom of getting it wrong, a hanging shutdown or a task that never
  stops.

## Related
- [[Concurrency MOC]]: the interrupt flag is how a pool asks a thread to stop, the thread-side
  counterpart of the data-locking notes there.
- [[A @Transactional timeout fails the next database access after the deadline instead of interrupting the method]]:
  that note shows a deadline that never interrupts the running method; this one covers an interrupt
  that is delivered, and what the `catch` must do with it.
- [[Java core MOC]]: the map entry for how exceptions interact with threads.
- Written up in win-interview: backend/java/docs/exceptions.md, section 2.2
