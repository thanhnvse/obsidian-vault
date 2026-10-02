---
tags: [system-design, interview, caching, redis]
status: draft
author: claude
up: ["[[Caching MOC]]"]
source: "https://redis.io/docs/latest/develop/reference/eviction/"
created: 2026-10-01
score: 0.862
review: "borderline"
score_reasons: ["atomic: 0.44 (borderline)", "numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 2
---
# Redis evicts keys only at maxmemory, and maxmemory 0, the 64-bit default, sets no limit at all

## Core idea
In Redis, eviction is a response to a memory limit, not to age: Redis evicts keys only when the memory used exceeds the maxmemory limit, choosing them by the configured maxmemory-policy until usage is back below the limit. Setting maxmemory to 0 means the memory for the dataset is not limited, and that is the default on 64-bit systems. A cache instance whose maxmemory was never set therefore has no limit to evict against, whatever policy is configured: keys leave only when they expire or are deleted, and memory keeps growing with every key written without a TTL.

## Why choose / why not
- Set maxmemory with allkeys-lru when: the instance is a pure cache whose entries can all be reloaded; Redis calls allkeys-lru a good default when a subset of keys is accessed far more often than the rest. Leave some RAM free for the replication and AOF buffers, which maxmemory does not count.
- Use a volatile-* policy only when: one instance holds both cache keys with TTLs and persistent keys without; those policies consider only keys with a TTL and behave like noeviction when none has one, and Redis suggests two separate instances instead if possible.
- Keep noeviction when: Redis holds data that must not silently disappear, such as sessions or a work queue; commands that add data then fail with an error at the limit instead of dropping keys.

## Interview angle
- Probed as "the hit rate dropped; is it the TTLs or eviction?"; compare evicted_keys with expired_keys in INFO stats.
- Common wrong answer: "Redis evicts old keys when it gets full", about an instance where maxmemory was never set.
- Strong answer: name the two mechanisms, set maxmemory and a policy on every cache instance, and read the two counters to tell them apart.

## Related
- [[In Redis a plain SET on an existing key clears its TTL]]: a key that silently lost its TTL is invisible to the volatile-* policies, so under them it can neither expire nor be evicted.
- [[LinkedHashMap in access order with removeEldestEntry is an LRU cache]]: the in-process form of the same rule; an LRU policy only evicts once something sets a bound, here removeEldestEntry, in Redis maxmemory.
