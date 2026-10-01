---
tags: [moc, java, collections, interview]
type: moc
status: draft
author: claude
up: ["[[Java core MOC]]"]
created: 2026-09-30
---
# Java collections MOC

The question behind this map: *which collection, and what contract does it rely on?*

## Collections
- [[ArrayList is the default List because LinkedList walks its nodes on every indexed access]]: the default choice, and why the "LinkedList for inserts" answer is wrong
- [[ArrayDeque replaces both Stack and LinkedList for stacks and queues]]: what to use when you really need a queue or a stack
- [[Overriding equals without hashCode makes HashMap lookups miss]]: the contract every hash-based collection relies on
- [[TreeMap trades HashMap's constant time for sorted keys and range queries]]: when ordering is worth O(log n)
- [[LinkedHashMap in access order with removeEldestEntry is an LRU cache]]: the classic follow-up to the Map comparison
