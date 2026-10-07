---
tags: [java, concurrency, collections, race-condition, interview]
status: draft
author: claude
up: ["[[Concurrency MOC]]"]
source: "https://docs.oracle.com/en/java/javase/23/docs/api/java.base/java/util/concurrent/ConcurrentHashMap.html"
created: 2026-10-07
review: unjudged
---
# A ConcurrentHashMap check-then-act is atomic only through computeIfAbsent, compute or merge, and their function runs inside the bin lock

## Core idea
Each `ConcurrentHashMap` call is atomic, but two calls are not: with `containsKey` then `put`, two
threads can both see "absent", both build the value, and one result overwrites the other.
`putIfAbsent` makes the check and the act one step for a value you already built. `computeIfAbsent`,
`compute` and `merge` do it for a value your function builds, by running the function inside
`synchronized` on the bin's first node (a placeholder node when the bin was empty). Two threads calling `computeIfAbsent` for
the same absent key run the function once, and the second returns the first one's value. The cost
is that other writers of that bin stall while your function runs, and so does any
thread whose resize must move that bin. The function must not write to the same map: JDK 23 throws
`IllegalStateException: Recursive update` only for the cases it detects, and JDK 8 could hang.

## Why choose / why not
- Use `computeIfAbsent` or `merge` when: the check and the act touch one key, such as a per-key
  cache entry or a counter. Keep the function short and free of I/O.
- Store a future instead of the value when: the load is slow, such as an HTTP call or a database
  query. The lock then covers only creating the future, and the slow load runs outside it.
- Don't use `compute` to enforce a map-wide limit when: other threads keep inserting into other
  bins. Reserve a slot first with an `AtomicInteger` or `Semaphore.tryAcquire()`.
- Don't wrap the calls in `synchronized (map)`: the map never uses its own monitor, so that block
  excludes nobody.

## Interview angle
- Probed as "is `containsKey` then `put` safe on a `ConcurrentHashMap`, and how would you build a
  cache on it?"
- Common wrong answers: "every operation is atomic, so code that uses it is thread-safe", and
  "`computeIfAbsent` is a non-blocking cache loader."
- Strong answer: each call is atomic, the pair is not; use `computeIfAbsent` or `merge`, keep the
  function short and never touch the map from it; for slow loads store a future, or use a cache
  library when entries need eviction.

## Related
- [[Concurrency MOC]]: the in-memory form of its question; the check and the act must happen under
  one lock.
- [[volatile makes count++ visible to other threads but does not make it atomic]]: the same
  read-modify-write window on a field; on a map entry, `compute` and `merge` close it per key.
- [[ConcurrentHashMap since JDK 8 locks the first node of one bin per write, not a segment, and get takes no lock]]:
  the bin lock this note relies on.
- [[On JDK 21 to 23 a virtual thread that blocks inside synchronized stays pinned to its carrier thread, and JDK 24 removes that]]:
  a slow function inside the bin lock is the blocking-in-`synchronized` case that pins there.
- Written up in win-interview: backend/java/docs/concurrent-maps.md, sections 2.3 and 2.6
