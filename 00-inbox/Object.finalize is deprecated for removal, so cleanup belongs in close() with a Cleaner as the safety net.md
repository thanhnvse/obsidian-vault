---
tags: [java, java-core, garbage-collection, interview]
status: draft
author: claude
up: ["[[Java core MOC]]"]
source: "https://openjdk.org/jeps/421"
created: 2026-10-01
score: 0.843
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Object.finalize is deprecated for removal, so cleanup belongs in close() with a Cleaner as the safety net

## Core idea
JEP 421, delivered in JDK 18, deprecated finalization for removal in a future release, and in JDK 21
`Object.finalize` carries `@Deprecated(since="9", forRemoval=true)`. The first flaw the JEP lists
is unpredictable latency: an arbitrarily long time may pass between an object becoming unreachable
and its finalizer running, and the GC gives no guarantee that any finalizer will ever be called. A
finalizer therefore cannot be relied on to release a resource, so cleanup moves to code that runs at
a known point. The
replacements the JEP and the `Object.finalize` Javadoc name are a `close()` method on an
`AutoCloseable` used with try-with-resources, and `java.lang.ref.Cleaner` for resources whose
lifetime outlives one block.

## Why choose / why not
- Implement `AutoCloseable` and use try-with-resources when: the code that opens a file, socket or
  native handle can also close it; the resource is released at the end of the block, even on an
  exception.
- Register a `Cleaner` action when: an object's lifetime cannot be tied to one block, such as a
  wrapper around native memory; the action runs some time after the object becomes unreachable, so
  treat it as a safety net behind `close()`, not as the main release path.
- Don't write a new `finalize` method when: you need any cleanup at all; it may run late or never,
  and it will stop running once finalization is disabled by default.

## Interview angle
- Probed as "how do you make sure a resource is released?" or "what is wrong with `finalize`?"
- Common wrong answer: "override `finalize` to close the resource, because the GC calls it."
- Strong answer: `finalize` is deprecated for removal (JEP 421) because it has no timing guarantee,
  can resurrect objects and runs on unspecified threads; use try-with-resources first and a `Cleaner`
  only as a backstop. Test with `--finalization=disabled` to find code that still depends on it.

## Related
- [[A Java memory leak is an object that stays reachable after the program stops needing it]]: a
  finalizer that stores `this` makes its object reachable again, which turns cleanup code into the
  kind of leak that note describes.
- [[Spring calls a public close() or shutdown() on a @Bean object at shutdown unless destroyMethod is empty]]:
  in a Spring application the container calls that explicit `close()` for singleton beans, so
  cleanup does not have to wait for the garbage collector.
