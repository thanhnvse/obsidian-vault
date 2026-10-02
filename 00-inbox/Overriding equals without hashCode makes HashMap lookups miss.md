---
tags: [java, java-core, collections, interview]
status: draft
author: claude
source: "https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Object.html"
created: 2026-09-30
score: 0.89
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Overriding equals without hashCode makes HashMap lookups miss

## Core idea
The `Object.hashCode` contract says that two objects equal by `equals` must return the same hash
code, while unequal objects may share one. `HashMap` uses the key's hash to choose a bucket and
then `equals` to find the key inside that bucket, which is why `get` and `put` are constant time
only when the hash function spreads keys across the buckets. If a class overrides `equals` but
keeps `Object.hashCode`, two equal but distinct instances usually get different hash codes, so
`get` with an equal key looks in the wrong bucket and returns `null`, without any exception.

## Why choose / why not
- Override `equals` and `hashCode` together when: instances are used as `HashMap` keys or
  `HashSet` elements; build both from the same fields.
- Use a `record` instead when: the key is a plain value; a record's implicit `equals` and
  `hashCode` are derived from its components, so they cannot drift apart.
- Override neither when: identity is the right equality, as for a connection or a listener;
  the inherited `Object` pair is already consistent.

## Interview angle
- Probed as "what breaks if I override only `equals`?"; the answer is a silent lookup miss, then
  duplicates in a `HashSet`.
- Common wrong answer: "`hashCode` must be unique for every object." Only equal objects must
  share a hash code; unique codes just make the table faster.
- Strong answer: state the direction of the contract (equal objects, equal hash codes, never the
  reverse), then walk through the bucket-then-`equals` lookup to show where the miss happens.

## Related
- [[Java collections MOC]]: the equals and hashCode contract is the core question behind
  every map and set in the Java core collections cluster.
