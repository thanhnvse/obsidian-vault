---
tags: [system-design, interview, sharding, database, scaling]
status: draft
author: claude
up: ["[[System design MOC]]"]
source: "https://learn.microsoft.com/en-us/azure/architecture/patterns/sharding"
created: 2026-10-01
score: 0.897
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Routing rows by hash(key) mod N moves most keys when a shard is added

## Core idea
Hash-based sharding computes each item's shard from a hash of its shard key, which spreads sequential key values across shards and needs no lookup map. With a standard function such as hash(key) mod N, the shard of almost every key depends on N, so adding or removing one shard reassigns most keys and turns a capacity change into a large-scale data migration. Microsoft's Sharding pattern states this cost directly: rebalancing a hash-sharded store is difficult without consistent hashing, which arranges the hash space so that only a small fraction of keys move when the shard count changes.

## Why choose / why not
- Choose consistent hashing when: shards are added or removed while the system runs; only the keys next to the changed shard move.
- Choose many virtual shards on few servers when: you own the routing layer; the key-to-virtual-shard function never changes, only the virtual-to-physical map, as in Microsoft's example of 11 logical shards on 3 databases (example numbers).
- Accept plain mod N only when: the shard count is fixed for the system's lifetime, or a full migration with downtime is acceptable each time it changes.

## Interview angle
- Probed as "you shard by hash(customer_id) mod 4 and traffic doubles; what happens when you add a fifth shard?"
- Common wrong answer: "only a fifth of the data moves"; with mod N most keys change shard, so the resize is a full migration.
- Strong answer: name the rehash problem, then consistent hashing or a fixed, large number of virtual shards, decided before the first row is written.

## Related
- [[Sharding makes cross-shard operations expensive, so the shard key must keep most work on one shard]]: that note chooses what to shard on; this one is about the function that maps the key to a shard, which decides what a later resize costs.
- [[Adding nodes to a Redis Cluster cannot relieve a single hot key]]: Redis Cluster maps keys to a fixed set of hash slots that are assigned to nodes, a form of virtual sharding: adding a node moves slots instead of rehashing every key.
