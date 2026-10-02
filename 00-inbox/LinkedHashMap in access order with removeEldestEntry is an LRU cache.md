---
tags: [java, java-core, collections, caching, interview]
status: draft
author: claude
source: "https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/LinkedHashMap.html"
created: 2026-09-30
score: 0.912
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 2
---
# LinkedHashMap in access order with removeEldestEntry is an LRU cache

## Core idea
`LinkedHashMap` is a hash table that also keeps a doubly-linked list running through its entries,
and by default that list follows insertion order. The constructor
`LinkedHashMap(initialCapacity, loadFactor, true)` switches it to access order, from
least-recently to most-recently accessed, where `get`, `put` and the other access methods move
the entry to the end. Overriding the protected `removeEldestEntry` method to return
`size() > MAX_ENTRIES` makes `put` drop the least-recently used entry after each insertion. The
Javadoc itself says this kind of map is well-suited to building LRU caches.

## Why choose / why not
- Choose it when: one thread needs a small bounded in-process LRU, such as memoising parsed
  templates inside a batch job; it takes about ten lines and no dependency.
- Don't choose it when: request threads share the cache; in access order even `get` is a
  structural modification, so every read needs a lock. Use a concurrent caching library such
  as Caffeine instead.
- Don't choose it when: entries must also expire after a time; `removeEldestEntry` is called
  only by `put` and `putAll` after an insertion, so an idle cache never evicts anything.

## Interview angle
- Probed as "implement an LRU cache"; naming `LinkedHashMap` with access order and
  `removeEldestEntry` shows you know the JDK before you hand-roll a map plus a linked list.
- Common wrong answer: using the no-argument constructor, which keeps insertion order, so
  reading an entry does not protect it from eviction.
- Strong answer: write the three-argument constructor and the override, then say it is not
  thread-safe and why: in access order `get` reorders the list.

## Related
- [[Overriding equals without hashCode makes HashMap lookups miss]]: `LinkedHashMap` finds entries
  the same way `HashMap` does, so cache keys need the same `equals` and `hashCode` contract.
- [[TreeMap trades HashMap's constant time for sorted keys and range queries]]: `LinkedHashMap`
  gives an ordered map by insertion or access, while `TreeMap` orders by key; pick by which
  order the query needs.
