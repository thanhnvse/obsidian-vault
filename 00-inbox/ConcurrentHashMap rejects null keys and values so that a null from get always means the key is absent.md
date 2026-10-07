---
tags: [java, java-core, collections, concurrency, interview]
status: draft
author: claude
up: ["[[Java collections MOC]]"]
source: "https://docs.oracle.com/en/java/javase/23/docs/api/java.base/java/util/concurrent/ConcurrentMap.html"
created: 2026-10-07
review: unjudged
---
# ConcurrentHashMap rejects null keys and values so that a null from get always means the key is absent

## Core idea
On a `HashMap`, `get(k) == null` means either "absent" or "mapped to `null`", and only a second
call, `containsKey`, tells them apart. On a map that several threads share, that second call is a
check-then-act race: the answer can be stale before you use it. `ConcurrentHashMap` therefore
throws `NullPointerException` for a `null` key or value, even for `get(null)` and
`containsKey(null)`, and `ConcurrentSkipListMap` does the same for the same reason. Six of the
seven default methods of `ConcurrentMap` state that they assume `get()` returning `null` means the
key is absent. A function passed to `compute`, `computeIfAbsent` or `merge` that returns `null`
means "no mapping": `computeIfAbsent` stores nothing, `compute` and `merge` remove the mapping.

## Why choose / why not
- Store a sentinel value object when: you need "known to be absent" (a negative cache) in a
  `ConcurrentHashMap`, because `null` cannot be stored.
- Use a `HashMap` that one thread owns when: you need `null` keys or values; there the second
  `containsKey` call is safe, which is why `HashMap` can allow `null`.
- Expect the same from `Hashtable`: it rejects `null` too, while `Collections.synchronizedMap`
  accepts whatever the wrapped map accepts, so a wrapped `HashMap` takes `null`.
- Watch `putIfAbsent` on a `HashMap` that maps a key to `null`: the `Map` API treats a `null` value
  as absent, so it overwrites it.

## Interview angle
- Probed as "why does `ConcurrentHashMap` not allow `null`?"
- Common wrong answer: "it allows one `null` key, like `HashMap`"; it throws
  `NullPointerException`, even for `get(null)`.
- Strong answer: `null` from `get` must be unambiguous; with `null` values you need `containsKey`
  afterwards, which is a race on a shared map. Quote the `ConcurrentSkipListMap` Javadoc reason and
  add that the `ConcurrentMap` default methods rely on it.

## Related
- [[Java collections MOC]]: the contract difference between `HashMap` and its concurrent sibling.
- [[A ConcurrentHashMap check-then-act is atomic only through computeIfAbsent, compute or merge, and their function runs inside the bin lock]]:
  those methods rest on the same rule, which is why a function returning `null` means no mapping.
- Written up in win-interview: backend/java/docs/concurrent-maps.md, section 2.8
