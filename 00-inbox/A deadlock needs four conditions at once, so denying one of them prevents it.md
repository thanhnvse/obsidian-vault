---
tags: [java, concurrency, deadlock, locks, interview]
status: draft
author: claude
up: ["[[Concurrency MOC]]"]
source: "https://doi.org/10.1145/356586.356588"
created: 2026-10-07
review: unjudged
---
# A deadlock needs four conditions at once, so denying one of them prevents it

## Core idea
A deadlock needs four conditions at once (Coffman, Elphick and Shoshani, 1971): mutual exclusion (a
lock has one owner), hold and wait (T1 keeps A while it waits for B), no preemption (only T1's
`unlock()` releases A) and circular wait (T1 waits for T2's B while T2 waits for T1's A).
`synchronized` and `ReentrantLock.lock()` satisfy all four by design, so two transfers that lock the
same two accounts in opposite order deadlock. Denying one condition prevents it, and the paper's
three approaches (Havender) map to Java: take every lock at once, one coarser lock (hold and wait);
give back what you hold when a request fails, `tryLock` with a timeout (no preemption); request
locks in one linear order, a global lock order such as by account id (circular wait).

## Why choose / why not
- Choose a global lock order when: you have a stable key, such as the account id, and every code
  path follows it; it costs nothing at run time and the second thread only waits. It fails when
  locks are taken inside callbacks or libraries, so do not call code you do not control while
  holding a lock.
- Choose `tryLock` with a timeout on the second lock when: a request may fail fast; release the
  first lock on failure. At least one side backs off, the caller needs a failure path, and a retry
  needs a random pause, or the two sides can keep colliding (a livelock).
- Choose one coarser lock when: the lost concurrency is acceptable, because one lock cannot form a
  cycle.
- Choose `lockInterruptibly()` with a canceller, such as a request timeout or shutdown, when: a
  stuck wait must end without a restart; threads deadlocked on `synchronized` monitors ignore an
  interrupt, and only a restart frees them.

## Interview angle
- Probed as "what is a deadlock, and how do you prevent it?"
- Common wrong answer: "deadlocks are a database problem, and the database resolves them"; Java
  threads deadlock on monitors and locks with no database, and nothing in the JVM breaks the cycle.
- Strong answer: name the four conditions, give one fix per condition, say you would pick lock
  ordering first because it costs nothing at run time, then say how to find one: a thread dump
  from `jcmd <pid> Thread.print -l` has a "Found one Java-level deadlock" section.

## Related
- [[Concurrency MOC]]: it is the thread-level hazard in the map of what goes wrong when work touches
  the same data at once.
- [[A deadlock aborts the whole transaction in PostgreSQL but only one statement in Oracle]]: the
  database has the same cycle, but it detects it and rolls back a victim, while the JVM can report
  a thread deadlock and cannot end it.
- [[SELECT FOR UPDATE makes competing writers queue on the row until the lock holder commits]]: row
  locks taken in opposite order form the same circular wait in SQL, and one consistent order avoids it.
- Written up in win-interview: backend/java/docs/locks-and-thread-hazards.md, sections 2.8 and 3.4
