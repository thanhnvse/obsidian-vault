---
tags: [java, java-core, streams, interview]
status: draft
author: claude
up: ["[[Java core MOC]]"]
source: "https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html#toMap(java.util.function.Function,java.util.function.Function)"
created: 2026-09-30
score: 0.833
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Collectors.toMap throws on a duplicate key unless you pass a merge function

## Core idea
The two-argument `Collectors.toMap(keyMapper, valueMapper)` lets each key appear only once. Its
JDK 21 Javadoc says that if the mapped keys contain duplicates according to `equals`, an
`IllegalStateException` is thrown when the collection operation is performed; on OpenJDK 21 the
message names the key, for example `Duplicate key a`. The three-argument overload takes a merge
function that decides which value wins when two elements map to the same key, so a duplicate no
longer fails the stream.

## Why choose / why not
- Use the two-argument form when: the key is unique by construction, such as a primary key, and a
  duplicate would be a bug that should fail loudly.
- Pass a merge function when: real data may repeat a key, such as customers keyed by email domain,
  and one value per key is enough.
- Don't use `toMap` when: every value for a key must be kept, such as all orders per customer;
  `groupingBy` collects them into a list per key instead.

## Interview angle
- Probed in code review: "what happens here if two orders have the same id?"
- Common wrong answer: "the last one wins, like `Map.put`."
- Strong answer: `IllegalStateException` at the terminal operation, the merge-function overload,
  and `groupingBy` when all values must survive.

## Related
- [[Stream intermediate operations are lazy and run only when a terminal operation starts]]: the
  exception surfaces at `collect`, the terminal operation, not where the mappers are declared.
- [[Overriding equals without hashCode makes HashMap lookups miss]]: a duplicate means a key equal by
  `equals`, so a key class with broken `equals` or `hashCode` changes what counts as a duplicate.
