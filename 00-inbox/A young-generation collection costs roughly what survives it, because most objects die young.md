---
tags: [java, java-core, garbage-collection, interview]
status: draft
author: claude
up: ["[[Java core MOC]]"]
source: "https://docs.oracle.com/en/java/javase/21/gctuning/garbage-collector-implementation1.html"
created: 2026-10-01
score: 0.888
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 2
---
# A young-generation collection costs roughly what survives it, because most objects die young

## Core idea
The HotSpot GC tuning guide for JDK 21 says HotSpot's collectors use generational collection to
exploit the weak generational hypothesis, which states that most objects survive for only a short
period of time. Most objects are allocated in the young generation and most die there; when it fills
up, a minor collection collects only the young generation. The guide says the cost of such a
collection is, to the first order, proportional to the number of live objects being collected, so a
young generation full of dead objects is collected very quickly. Some of the survivors move to the
old generation at each minor collection, and when the old generation fills up a major collection
covers the whole heap and usually lasts much longer than a minor one.

## Why choose / why not
- Allocate short-lived objects freely when: they die within a request, such as DTOs, iterators or
  builders; a young collection pays for the few that survive, not for the many that were allocated.
- Don't keep objects alive "for a while" without a reason when: a cache entry lives for minutes or a
  large batch is held across requests; such objects survive young collections, are copied and
  promoted, and are reclaimed only by old-generation work.
- Don't pool ordinary objects to "help the GC" when: the objects are cheap to create; a pool keeps
  them alive into the old generation, which is the expensive case.

## Interview angle
- Probed as "what is the generational hypothesis, and why does it matter?"
- Common wrong answer: "the collector scans every object in the heap each time, so allocating less is
  always faster."
- Strong answer: most objects die young, so a young collection costs what survives rather than what
  was allocated; the expensive objects are the middle-aged ones that get promoted and die in the old
  generation.

## Related
- [[A Java memory leak is an object that stays reachable after the program stops needing it]]: that
  note defines a leak as an object that stays reachable; such an object is the opposite of one that
  dies young, so it survives every minor collection and ends up in the old generation, where it costs
  the most.
- [[In JDK 21 ZGC is single-generation unless the ZGenerational flag is set]]: this note explains why
  a young generation pays off; that note shows that ZGC on JDK 21 has one only when the
  `ZGenerational` flag is set.
