---
tags: [java, concurrency, threads, conditions, interview]
status: draft
author: claude
up: ["[[Concurrency MOC]]"]
source: "https://docs.oracle.com/javase/specs/jls/se21/html/jls-17.html#jls-17.2.1"
created: 2026-10-07
review: unjudged
---
# A wait() or await() call belongs in a while loop, because a woken thread can find its condition false again

## Core idea
`wait()` and `Condition.await()` release the lock while they wait and take it back, with the same
hold count, before they return. `signal()` and `notify()` do not hand the lock over: the woken
thread must still get the lock, so it returns only after the signaller unlocks, and another thread
can change the state in between. In the lab a consumer is signalled, a second consumer takes the
item first, and the woken one finds the queue empty again: with `if` it removes from an empty queue
and throws `NoSuchElementException`, with `while` it checks again and waits again. The JLS also
allows a spurious wakeup, a return with no signal, interrupt or timeout, and concludes that `wait`
belongs only inside loops that end when the awaited condition holds.

## Why choose / why not
- Write `while (!ready) condition.await();` when: threads wait for state that another thread
  changes, such as a bounded buffer; one loop covers both the stolen item and the spurious wakeup.
- Choose a `Condition` over `wait`/`notify` when: several kinds of waiter share one lock, such as
  producers and consumers. `signal()` wakes only the waiters of that condition, while `notify()` on
  a monitor can wake the wrong kind, which goes back to sleep while the right one keeps sleeping;
  monitor code then uses `notifyAll()` and wakes everyone.
- Don't hand-write it when: a ready-made queue fits; `ArrayBlockingQueue` already does it, and the
  `Condition` javadoc ends its bounded-buffer example by pointing to it.
- Hold the lock before you wait or signal: otherwise `IllegalMonitorStateException`, and checking
  the state and going to sleep would no longer be one step.

## Interview angle
- Probed as "`wait`/`notify` vs `Condition`? Why the `while` loop?"
- Common wrong answer: "`if (queue.isEmpty()) wait();` is fine, nobody else calls `notify`";
  another consumer can take the item first, and spurious wakeups are allowed.
- Strong answer: both release the lock while waiting and get it back before returning, so the woken
  thread may find the condition false again; the JLS allows spurious wakeups; `Condition` adds
  several wait sets per lock and timed or interruptible waits.

## Related
- [[Concurrency MOC]]: it is the thread-coordination half of the map, next to the notes on locks
  and data races.
- [[A ReentrantLock needs lock() just before the try block and unlock() in finally, or an exception leaks the lock or hides the real error]]:
  a `Condition` comes from a `Lock` such as a `ReentrantLock`, and `await()` returns holding that
  lock again, so the same release idiom applies to the section around the loop.
- Written up in win-interview: backend/java/docs/locks-and-thread-hazards.md, section 2.7
