---
tags: [java, java-core, jvm, gc, interview]
status: draft
author: claude
source: "https://docs.oracle.com/en/java/javase/21/troubleshoot/troubleshooting-memory-leaks.html"
created: 2026-09-30
score: 0.827
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A Java memory leak is an object that stays reachable after the program stops needing it

## Core idea
The HotSpot garbage collector treats an object as garbage only when it can no longer be reached
from any reference of any other live object, so an object that the program no longer needs but
still references is never collected. A Java memory leak is therefore a reachability bug: the
Oracle troubleshooting guide calls an application unintentionally holding references that
prevent garbage collection the Java language equivalent of a memory leak. Such objects can grow
over time until they fill the heap and the process fails with `java.lang.OutOfMemoryError`.

## Why choose / why not
- Suspect a leak when: heap use keeps climbing over hours of steady load and collections get more
  frequent; the set of reachable objects is growing, not just the allocation rate.
- Don't assume a leak when: one `OutOfMemoryError: Java heap space` appears; the guide says it
  does not necessarily imply a leak, and the heap may simply be too small, which a larger `-Xmx`
  fixes.
- Fix the reference, not the collector: bound the cache, remove the entry, or call
  `ThreadLocal.remove()` in a `finally` block on pooled threads; no collector and no `System.gc()`
  call frees an object that is still reachable.

## Interview angle
- Probed as "can a Java program leak memory when it has a garbage collector?"; the interviewer
  wants the reachability definition, not "no".
- Common wrong answer: "the GC handles it", or "call `System.gc()`".
- Strong answer: define the leak by reachability, give one cause you have met, then find what
  keeps the growing objects reachable in a heap dump.

## Related
- [[ZGC trades some throughput for sub-millisecond pauses, so G1 stays the default collector]]:
  collector choice changes pause times, but no collector frees a reachable object, so a leak
  survives a switch to ZGC.
- [[Python memory is managed by reference counting plus a cycle collector]]: CPython needs a
  separate cycle collector, while a tracing collector frees unreachable cycles anyway; in both
  runtimes an object that is still referenced stays in memory.
- [[JVM MOC]]: memory leaks sit next to garbage collection on the JVM topic map.
