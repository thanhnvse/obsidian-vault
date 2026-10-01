---
tags: [system-design, caching, redis, interview]
status: draft
author: claude
up: ["[[Caching MOC]]"]
source: "https://redis.io/docs/latest/operate/oss_and_stack/reference/cluster-spec/"
created: 2026-10-01
score: 0.885
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Adding nodes to a Redis Cluster cannot relieve a single hot key

## Core idea
Redis Cluster splits the key space into 16384 hash slots and maps every key to one of them with HASH_SLOT = CRC16(key) mod 16384. When the cluster is stable, a single hash slot is served by a single node. Every request for one key therefore reaches the same primary however many nodes the cluster has; Redis's anti-patterns guide describes a 99-node cluster in which a million requests per second for one key all go to one node. Adding nodes spreads different keys, not one key, so the fixes change the key or move reads off that node: write the value under several keys that hash to different slots, read from replicas with READONLY where stale reads are acceptable, or keep a short-lived local copy in each application instance.

## Why choose / why not
- Split a hot key into several copies when: one key dominates reads; every write must then update all copies.
- Read from replicas with READONLY when: slightly stale reads are acceptable; it adds read capacity without changing keys.
- Don't add Redis nodes when: the load is one key; the extra nodes stay idle while one primary saturates.

## Interview angle
- Probed as "we added Redis nodes and latency didn't move; why?"
- Common wrong answer: "scale the cluster out."
- Strong answer: explain slot placement, find the key (redis-cli --hotkeys works with an LFU eviction policy), then split it or cache it locally.

## Related
- [[A cache stampede happens when a hot key expires and many requests regenerate it at once]]: a hot key's expiry causes a stampede; this note is about the steady load on its single node.
- [[Sharding makes cross-shard operations expensive, so the shard key must keep most work on one shard]]: the same placement-by-key rule, seen from the database side.
