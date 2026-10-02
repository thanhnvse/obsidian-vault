---
tags: [java, java-core, jvm, gc, interview]
status: draft
author: claude
source: "https://docs.oracle.com/en/java/javase/25/gctuning/available-collectors.html"
created: 2026-09-30
score: 0.863
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# ZGC trades some throughput for sub-millisecond pauses, so G1 stays the default collector

## Core idea
G1, the default collector on server-class machines since JDK 9 (JEP 248), is a mostly concurrent
collector built to meet a pause-time goal with high probability while keeping throughput high;
the goal is `-XX:MaxGCPauseMillis`, 200 ms by default. ZGC, a product feature since JDK 15
(JEP 377) and enabled with `-XX:+UseZGC`, keeps maximum pause times under a millisecond,
independent of heap size, at the cost of some throughput, according to the JDK 25 GC tuning
guide. The choice between the two is therefore a trade: G1 balances latency and throughput,
while ZGC puts latency first.

## Why choose / why not
- Keep G1 when: the service meets its latency target with default settings; it needs little
  tuning and is what the JVM selects anyway.
- Choose ZGC when: tail latency matters more than peak throughput, such as an API whose p99
  target is broken by G1 pauses on a large heap.
- Don't choose ZGC when: throughput per CPU is what matters, as in an overnight batch job; there
  the guide points to the Parallel collector instead.

## Interview angle
- Probed as "which GC does your service run, and why?"; the interviewer wants the trade, not a
  favourite collector.
- Common wrong answer: "ZGC has no pauses." Its pauses are short, not absent, and it pays for
  them in throughput.
- Strong answer: name the JDK version and the collector the JVM picked, then the pause
  percentile that would justify moving to ZGC.

## Related
- [[JVM MOC]]: collector choice is the first garbage-collection topic on the JVM map.
- [[Python memory is managed by reference counting plus a cycle collector]]: CPython frees most
  objects the moment their count reaches zero, while HotSpot finds garbage by tracing which
  objects are still reachable, which is where collector pauses and this trade come from.
- [[Java core MOC]]: this is the garbage-collection entry in the Java core section.
