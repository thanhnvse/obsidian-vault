---
tags: [java, java-core, collections, interview]
status: draft
author: claude
source: "https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ArrayDeque.html"
created: 2026-09-30
score: 0.88
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# ArrayDeque replaces both Stack and LinkedList for stacks and queues

## Core idea
`ArrayDeque` is a resizable-array implementation of the `Deque` interface with no capacity
restrictions. Its Javadoc says it is likely to be faster than `Stack` when used as a stack, and
faster than `LinkedList` when used as a queue. The `Deque` and `Stack` Javadocs both say that
`Deque` should be used in preference to the legacy `Stack` class, which extends the synchronized
`Vector`; used as a stack, `push` and `pop` map to `addFirst` and `removeFirst`. `ArrayDeque`
prohibits `null` elements and is not thread-safe.

## Why choose / why not
- Choose `ArrayDeque` when: one thread owns a stack or a FIFO work list, such as the frontier of
  a depth-first or breadth-first walk, or an undo history; declare it as `Deque<T>`.
- Don't choose it when: several threads share the queue; use a `BlockingQueue` implementation
  such as `ArrayBlockingQueue`, or `ConcurrentLinkedDeque`, instead.
- Don't choose it when: the queue must hold `null`; `ArrayDeque` rejects it, so either pick a
  real sentinel object or, if `null` is unavoidable, `LinkedList`.

## Interview angle
- Probed as "how do you implement a stack in Java?"; answering `java.util.Stack` signals dated
  knowledge, because its own Javadoc points to `Deque`.
- Common wrong answer: "`LinkedList` is the standard queue because it implements `Queue`."
- Strong answer: `Deque<Integer> stack = new ArrayDeque<>()`, then cite the Javadoc remark that
  it is faster than `Stack` as a stack and faster than `LinkedList` as a queue.

## Related
- [[ArrayList is the default List because LinkedList walks its nodes on every indexed access]]:
  the same array-over-nodes argument applied to the `List` side, which is why `LinkedList` loses
  on both fronts.
- [[Java collections MOC]]: this is the follow-up to the list question in the Java core
  collections cluster.
