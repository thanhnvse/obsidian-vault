---
tags: [java, java-core, collections, concurrency, interview]
status: draft
author: claude
up: ["[[Java collections MOC]]"]
source: "https://github.com/openjdk/jdk23u/blob/jdk-23.0.2-ga/src/java.base/share/classes/java/util/concurrent/ConcurrentHashMap.java#L268-L497"
created: 2026-10-07
review: unjudged
---
# ConcurrentHashMap since JDK 8 locks the first node of one bin per write, not a segment, and get takes no lock

## Core idea
Since JDK 8 `ConcurrentHashMap` is one array of bins, like `HashMap`, and the lock granularity
is one bin. A `put` into an empty bin is a single CAS with no lock; for a bin that holds nodes
the writer takes `synchronized` on the bin's first node, rechecks that it is still the first node,
then replaces the value or appends. `get` takes no lock: a node's `hash` and `key` are `final` and
its `val` and `next` are `volatile`, so a reader always sees a complete node. JDK 7 instead split
the map into segments, 16 by default, each a `ReentrantLock`. Since JDK 8 `concurrencyLevel` only
raises the initial table size: `new ConcurrentHashMap<>(16, 0.75f, 64)` gets 128 bins, no segments.

## Why choose / why not
- Choose it when: a mutable map is shared between threads, such as sessions or a per-tenant
  cache. Reads never wait, and writers meet only when they hit the same bin.
- Don't assume the lock is per key when: two keys land in one bin. `"Aa"` and `"BB"` share hash
  code 2112, so a slow `compute` on one makes writers of the other wait. Readers are never blocked.
- Use `Collections.synchronizedMap` when: one lock over the whole map is what you want, so a caller
  can make several calls atomic with `synchronized (map)`. On a `ConcurrentHashMap` that block
  excludes nobody, because the map never uses its own monitor.
- Don't use it as a cache that must stay bounded when: entries need eviction or expiry; the map has
  none, so it only grows → use a cache library.

## Interview angle
- Probed as "how does `ConcurrentHashMap` work internally?" and "what changed in Java 8?"
- Common wrong answers: "it locks segments", or "reads are synchronised too, on a smaller lock."
- Strong answer: CAS into an empty bin, the first node's monitor with a recheck otherwise, a
  lock-free `get`; then derive who waits for whom (writers of the same bin, and a resize that must
  move a locked bin) and say that `concurrencyLevel` is only a sizing hint.

## Related
- [[Java collections MOC]]: the thread-safe member of the Map family, built like `HashMap` with a
  lock per bin.
- [[Concurrency MOC]]: the in-memory version of its question, what happens when two threads touch
  the same entry at once.
- [[A HashMap bin turns into a red-black tree only once the table has at least 64 buckets]]:
  `ConcurrentHashMap` uses the same thresholds (8, 6 and 64), with a `TreeBin` wrapper as the head
  of a tree bin.
- [[Overriding equals without hashCode makes HashMap lookups miss]]: colliding keys share one bin
  lock here, so a poor `hashCode` costs concurrency as well as lookups.
- Written up in win-interview: backend/java/docs/concurrent-maps.md, sections 2.2 and 2.9
