---
tags: [java, java-core, garbage-collection, interview]
status: draft
author: claude
up: ["[[Java core MOC]]"]
source: "https://docs.oracle.com/en/java/javase/21/gctuning/garbage-first-garbage-collector-tuning.html"
created: 2026-10-01
score: 0.869
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# On G1, System.gc() triggers a stop-the-world full collection by default

## Core idea
The JDK 21 `System.gc()` Javadoc says that calling it suggests that the JVM expend effort toward
recycling unused objects, with no guarantee about how much is reclaimed or when. The JDK 21 GC tuning
guide says an explicit `System.gc()` call can force a major collection when a minor one would
suffice, and that `-XX:+DisableExplicitGC` makes the VM ignore such calls. Its G1 tuning chapter
lists `System.gc()` as a cause of a Full GC whose effect `-XX:+ExplicitGCInvokesConcurrent` can
mitigate, and the JDK 21 `java` reference says that flag makes `System.gc()` invoke a concurrent GC. A G1 Full GC is a stop-the-world compaction of the whole heap. On OpenJDK
21 with G1, the unified GC log records such a call as `Pause Full (System.gc())`.

## Why choose / why not
- Don't call `System.gc()` in application code when: it is meant to "free memory now"; on G1 it pays
  for a full stop-the-world pause, and it cannot free anything that is still reachable.
- Set `-XX:+ExplicitGCInvokesConcurrent` when: a library you cannot change calls `System.gc()` and the
  full pauses show in the GC log; the call then invokes a concurrent GC instead of a full one.
- Set `-XX:+DisableExplicitGC` when: you have checked that nothing depends on the explicit calls,
  such as the periodic collections that RMI distributed garbage collection requests; the JVM then
  ignores them.

## Interview angle
- Probed as "what does `System.gc()` do, and should you call it?"
- Common wrong answer: "it frees memory immediately."
- Strong answer: it only suggests a collection; on G1 the default response is a full stop-the-world
  collection, it can be disabled or made concurrent with a flag, and the GC log shows it as
  `Pause Full (System.gc())`.

## Related
- [[A Java memory leak is an object that stays reachable after the program stops needing it]]: that
  note explains why no `System.gc()` call frees a reachable object, so a forced full pause does not
  fix a leak either.
- [[ZGC trades some throughput for sub-millisecond pauses, so G1 stays the default collector]]: G1 is
  the collector a default JDK 21 server runs, which is why this full pause is the usual effect of the call.
