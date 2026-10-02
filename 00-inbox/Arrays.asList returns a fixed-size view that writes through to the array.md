---
tags: [java, java-core, collections, interview]
status: draft
author: claude
up: ["[[Java collections MOC]]"]
source: "https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Arrays.html#asList(T...)"
created: 2026-10-01
score: 0.911
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Arrays.asList returns a fixed-size view that writes through to the array

## Core idea
The JDK 21 Javadoc says `Arrays.asList(array)` returns a fixed-size list backed by the specified array:
changes made to the array are visible in the list, and changes made to the list are visible in the
array. The list supports the optional `Collection` methods except those that would change its size,
so `set` works and writes into the array, while `add` and `remove` throw
`UnsupportedOperationException`. On OpenJDK 21 its class is `java.util.Arrays$ArrayList`, a private
nested class that is not `java.util.ArrayList` despite having the same simple name. Code that receives this
list as a plain `List` and later calls `add` fails at run time, not at compile time.

## Why choose / why not
- Use `Arrays.asList` when: you need a `List` view of an existing array, such as passing an array to
  an API that takes a `List`, or shuffling an array in place with `Collections.shuffle`, and the size
  will not change.
- Copy it with `new ArrayList<>(Arrays.asList(array))` when: the caller will add or remove elements;
  the copy is a real `java.util.ArrayList` and no longer writes through to the array.
- Use `List.of(...)` instead when: you want a fixed list literal that nobody can change; it rejects
  `set` too, and it disallows `null` elements, while `Arrays.asList` accepts them.

## Interview angle
- Probed as "why does `Arrays.asList(...).add(x)` throw?", or as a code review of a method that
  returns `Arrays.asList(...)` to callers that later add to it.
- Common wrong answer: "`Arrays.asList` returns an `ArrayList`, so it is fully mutable."
- Strong answer: it is a fixed-size view over the array: `set` writes through, `add` and `remove`
  throw, and its class is `Arrays$ArrayList`; copy into `new ArrayList<>(...)` when the size must change.

## Related
- [[ArrayList is the default List because LinkedList walks its nodes on every indexed access]]: that
  note is about the real resizable `java.util.ArrayList`, which `Arrays$ArrayList` resembles only in
  name and in its array-backed, constant-time `get`.
