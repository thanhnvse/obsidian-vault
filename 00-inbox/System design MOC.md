---
tags: [moc, system-design, interview]
type: moc
status: draft
author: claude
up: ["[[Java backend interview MOC]]"]
created: 2026-09-30
---
# System design MOC

The question behind this map: *which load dominates, and what gives way first?*

## Read-heavy vs write-heavy
- [[Read replicas scale reads but serve stale data while replication lags]]: scaling reads
- [[A write-heavy database scales by sharding, because replicas add only read capacity]]: scaling writes
- [[Batching writes raises throughput by paying the per-request overhead once per batch]]: cheaper writes before sharding

## Caching
- [[Cache-aside loads data on a miss and leaves cache consistency to the application]]: the default strategy
- [[Write-behind caching makes the cache the system of record until its queue is flushed]]: the fast and risky one
- [[A cache stampede happens when a hot key expires and many requests regenerate it at once]]: the failure mode to name

## Queues and scaling
- [[A queue in front of a service levels load spikes at the cost of an immediate response]]: why a queue, and what it costs
- [[Stateless services scale out, while scaling up one machine stops at a hardware limit]]: horizontal vs vertical
- [[Sharding makes cross-shard operations expensive, so the shard key must keep most work on one shard]]: choosing a shard key
