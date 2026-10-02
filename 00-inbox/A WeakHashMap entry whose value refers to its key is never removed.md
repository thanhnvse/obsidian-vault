---
tags: [java, java-core, collections, garbage-collection, interview]
status: draft
author: claude
up: ["[[Java collections MOC]]"]
source: "https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/WeakHashMap.html"
created: 2026-09-30
score: 0.867
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A WeakHashMap entry whose value refers to its key is never removed

## Core idea
`WeakHashMap` holds its keys through weak references, so an entry can disappear once nothing else
strongly references its key. Its values are ordinary strong references held by the map. If a value
refers to its own key, directly or through other objects, the map itself keeps the key strongly
reachable and the entry is never removed. The JDK 21 `WeakHashMap` Javadoc warns about this in its
implementation note, including the indirect case through another entry's value, and suggests
wrapping values in `WeakReference`s when the values do not rely on the map to keep them alive. On
OpenJDK 21, an entry that maps a key `user` to `new Session(user)` is still present, and the key
still reachable, after three `System.gc()` calls.

## Why choose / why not
- Use `WeakHashMap` when: attaching data to objects whose lifetime someone else owns, and the value
  does not point back at the key.
- Don't use it when: the value holds the key, such as a session object that stores the user it is
  keyed by; the "self-cleaning" map then grows forever.
- Don't use it as a general cache when: entries should go by size or age; keys vanish only when the
  key becomes unreachable, so use a cache with size or time eviction instead.

## Interview angle
- Probed as "how would you build a map that cleans itself up?"
- Common wrong answer: "use a `WeakHashMap`", with no word about what the values reference.
- Strong answer: weak keys, strong values, the value-to-key pin from the Javadoc, and that a weak
  map is not an LRU or a TTL cache.

## Related
- [[A Java memory leak is an object that stays reachable after the program stops needing it]]: that
  note defines a leak by reachability; here the path that keeps the key reachable is
  map → value → key, so the weak key never becomes only weakly reachable and the entry leaks.
- [[LinkedHashMap in access order with removeEldestEntry is an LRU cache]]: when the real goal is a
  bounded cache, that map evicts by size and recency on every `put`, which a `WeakHashMap` never
  does because it removes entries only when their keys become unreachable.
