---
tags: [java, java-core, garbage-collection, interview]
status: draft
author: claude
up: ["[[Java core MOC]]"]
source: "https://openjdk.org/jeps/439"
created: 2026-09-30
score: 0.914
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# In JDK 21 ZGC is single-generation unless the ZGenerational flag is set

## Core idea
JDK 21 delivered Generational ZGC (JEP 439) as an opt-in mode: `-XX:+UseZGC` alone still selects the
older non-generational ZGC, and `-XX:+UseZGC -XX:+ZGenerational` selects the generational one. On
OpenJDK 21.0.1 the two modes show different `GarbageCollectorMXBean`s: `ZGC Cycles` and
`ZGC Pauses` without the flag, `ZGC Minor Cycles` and `ZGC Major Cycles` with it, and
`-XX:+PrintFlagsFinal` reports `ZGenerational` as false by default. Generational mode became the
default in JDK 23 (JEP 474), and JDK 24 removed the non-generational mode (JEP 490).

## Why choose / why not
- Add `-XX:+ZGenerational` on JDK 21 when: you pick ZGC for latency; the JDK 21 GC tuning guide's own
  selection advice pairs `-XX:+UseZGC` with `-XX:+ZGenerational`.
- Don't assume the mode when: auditing a JDK 21 service; check the collector bean names or the
  startup flags instead of the `-XX:+UseZGC` line alone.
- Revisit the flag when: upgrading to JDK 23 or later, where generational mode is already the default.

## Interview angle
- Probed as "which garbage collector does your service run, and in which mode?"
- Common wrong answer: "ZGC is generational" for a JDK 21 service started with `-XX:+UseZGC` only.
- Strong answer: name the JDK version, the flag, and how you verified the mode (bean names or
  `-XX:+PrintFlagsFinal`).

## Related
- [[ZGC trades some throughput for sub-millisecond pauses, so G1 stays the default collector]]: that
  note is about choosing ZGC at all; this one is about which ZGC JDK 21 gives you once you do.
