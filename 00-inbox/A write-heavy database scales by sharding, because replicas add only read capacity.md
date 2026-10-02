---
tags: [system-design, interview, database, sharding, scaling]
status: draft
author: claude
source: "https://learn.microsoft.com/en-us/azure/architecture/patterns/sharding"
created: 2026-09-30
score: 0.885
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A write-heavy database scales by sharding, because replicas add only read capacity

## Core idea
With one primary and read replicas, clients write only to the primary and read from the
replicas, so adding replicas adds read capacity but no write capacity. To scale writes, sharding
divides the data store into horizontal partitions (shards): each shard has the same schema, holds
its own subset of the data and runs on its own server, so the load, writes included, is spread over
several servers. Microsoft's Sharding pattern names this trigger: throughput exceeds what one
instance can sustain, and read replicas alone don't resolve the bottleneck because write load is
also high. Scaling the single server up only postpones the problem, because it eventually reaches a
limit where compute and storage can't grow any further.

## Why choose / why not
- Choose sharding when: write throughput or data volume exceeds what one primary can sustain even
  after scaling it up, and most operations can be scoped to one shard key, such as a tenant ID.
- Don't shard when: the bottleneck is read volume; read replicas and caches offload reads without
  the cross-shard query complexity that sharding brings.
- Don't shard yet when: data and throughput fit one instance with projected growth; scaling up
  keeps queries simple and transactions intact.

## Interview angle
- Probed as "the primary is saturated by inserts; would you add read replicas?"; the expected answer
  is no, because replicas take no writes.
- Common wrong answer: reaching for replicas or a cache when the bottleneck is writes.
- Strong answer: rule out the cheaper levers first (scale up, fewer and larger writes), then choose
  the shard key from the dominant query pattern and state the price: queries that span shards.

## Related
- [[Read replicas scale reads but serve stale data while replication lags]]: the read-side
  counterpart; replicas stop helping exactly where this note starts, when writes rather than reads
  saturate the primary.
- [[System design MOC]]: this is the write-heavy answer to the System design cluster's
  question of which load dominates.
