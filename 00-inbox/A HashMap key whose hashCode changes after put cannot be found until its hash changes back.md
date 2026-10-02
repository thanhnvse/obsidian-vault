---
tags: [java, java-core, collections, interview]
status: draft
author: claude
up: ["[[Java collections MOC]]"]
source: "https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Map.html"
created: 2026-10-01
score: 0.867
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A HashMap key whose hashCode changes after put cannot be found until its hash changes back

## Core idea
The JDK 21 `Map` Javadoc warns that great care must be exercised if mutable objects are used as map
keys, because the behavior of a map is not specified if a key's value changes in a way that affects
`equals` comparisons while it is in the map. In OpenJDK 21, `HashMap` stores each entry in the bucket
chosen by the key's hash at `put` time and caches that hash in the node. If a field that `hashCode`
reads changes afterwards, `get` and `containsKey` compute the new hash, look in another bucket or fail
the cached-hash comparison, and miss. The entry is still in the table: `size()` counts it and
iteration returns it. Changing the field back makes the entry findable again.

## Why choose / why not
- Use immutable keys when: an object goes into a `HashMap` or `HashSet`; a `record` or a `String` id
  cannot change after `put`, so its hash cannot drift.
- Remove, change, then re-insert when: a key object must change; take the entry out before the
  mutation and `put` it again afterwards.
- Don't build `hashCode` from a JPA entity's generated id when: entities are put in a `Set` before
  they are persisted; the id is assigned on persist, so the hash changes while the entity is in the set.

## Interview angle
- Probed as "what happens if you change a `HashMap` key after inserting it?", often after the
  `equals` and `hashCode` question.
- Common wrong answer: "the map updates itself", or "`get` throws an exception."
- Strong answer: the node stays in the bucket of the old hash, so lookups silently miss while the
  entry still counts in `size()`; the `Map` contract leaves the behaviour unspecified, so keep keys
  immutable.

## Related
- [[Overriding equals without hashCode makes HashMap lookups miss]]: that note breaks the lookup with
  a wrong class contract; this one breaks it with a correct contract whose inputs change after `put`.
- [[A Java memory leak is an object that stays reachable after the program stops needing it]]: an
  entry that no lookup can find but the table still references is one of those leaks, and a map that
  keeps re-adding such keys grows without bound.
