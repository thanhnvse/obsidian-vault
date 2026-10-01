---
tags: [moc, java, java-core, interview]
type: moc
status: draft
author: claude
up: ["[[Java backend interview MOC]]"]
created: 2026-09-30
---
# Java core MOC

The question behind this map: *what does the JVM actually do with this code?*

## Collections
- [[Java collections MOC]]: the five collection choices and the equals/hashCode contract

## Lambdas and streams
- [[A lambda is linked through invokedynamic at run time, not compiled to its own class file]]: the mechanical difference from an anonymous class
- [[Inside a lambda, this refers to the enclosing instance, not to the lambda]]: the semantic difference interviewers test
- [[Stream intermediate operations are lazy and run only when a terminal operation starts]]: why a stream without a terminal operation does nothing
- [[Parallel streams share the JVM-wide common ForkJoinPool]]: why `parallel()` is not a free speed-up in a server

## Garbage collection
- [[ZGC trades some throughput for sub-millisecond pauses, so G1 stays the default collector]]: choosing a collector by pause goal
- [[A Java memory leak is an object that stays reachable after the program stops needing it]]: what "leak" means with a GC
