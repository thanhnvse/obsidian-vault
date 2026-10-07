---
tags: [java, concurrency, virtual-threads, jvm, interview]
status: draft
author: claude
up: ["[[Concurrency MOC]]"]
source: "https://openjdk.org/jeps/491"
created: 2026-10-07
review: unjudged
---
# On JDK 21 to 23 a virtual thread that blocks inside synchronized stays pinned to its carrier thread, and JDK 24 removes that

## Core idea
A virtual thread that blocks on I/O or on a JDK blocking operation such as a `ReentrantLock` or a
latch normally unmounts, and its carrier thread then runs another virtual thread. On JDK 21 to 23 a
virtual thread inside a `synchronized` block or method, or in a native frame, cannot unmount: the
JVM tracks which platform thread holds a monitor, not which virtual thread (JEP 491). It parks the
carrier itself and is pinned, and the scheduler does not add carriers to make up for it (JEP 444).
With the scheduler limited to one carrier, a virtual thread that waits on a latch inside
`synchronized` deadlocks against the thread that would release it (tested on 23.0.2). JDK 24 lets a
virtual thread unmount while it blocks in `synchronized`, and native frames still pin (JEP 491).

## Why choose / why not
- Replace `synchronized` with `ReentrantLock` when: the block does long blocking work, such as a
  database call, and the application runs on JDK 21 to 23; JEP 444 gives this
  advice.
- Leave `synchronized` alone when: the block never blocks, or the JDK is 24 or later. Pinning
  hurts scalability, not correctness (JEP 444).
- Detect it with the JFR event `jdk.VirtualThreadPinned` (20 ms threshold by default), which is
  recorded only when the pinned wait ends. For a stall that never ends use
  `-Djdk.tracePinnedThreads=full`, which prints when the thread blocks; JDK 24 removes that
  property.

## Interview angle
- Probed as "what is pinning?", usually right after "what are virtual threads and when would you
  use them?"
- Common wrong answer: "virtual threads make code faster" (they add throughput for waiting work,
  not speed).
- Strong answer: carrier, mount and unmount; the two pin cases on JDK 21 to 23 (`synchronized` and
  native frames); the effect (carriers taken from every other virtual thread, a deadlock with one
  carrier); JFR for waits that end and `tracePinnedThreads` for stalls; the fix by version.

## Related
- [[Concurrency MOC]]: virtual threads change what a blocked thread costs, so this is the limit
  to know before moving blocking code onto them.
- [[A ConcurrentHashMap check-then-act is atomic only through computeIfAbsent, compute or merge, and their function runs inside the bin lock]]:
  its function runs inside a `synchronized` monitor, so a slow call there pins a virtual thread on
  JDK 21 to 23.
- Written up in win-interview: backend/java/docs/thread-pools-and-async.md, section 2.8 (Pinning)
