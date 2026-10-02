---
tags: [system-design, interview, sharding, database, scaling]
status: draft
author: claude
source: "https://learn.microsoft.com/en-us/azure/architecture/patterns/sharding"
created: 2026-09-30
score: 0.88
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Sharding makes cross-shard operations expensive, so the shard key must keep most work on one shard

## Core idea
Once data is sharded, a query that needs data from several shards has to fan out to each shard in
parallel and aggregate the results, and even then the slowest shard sets the overall latency.
Transactions across shards are harder still: distributed coordination such as two-phase commit adds
latency and failure modes and reduces throughput, so most sharded systems avoid it and accept
eventual consistency. The shard key therefore has to follow the dominant query pattern, so that most
requests resolve against a single shard, for example by keeping a customer and their orders in the
same shard. Changing the key later typically means migrating all data to a new shard layout.

## Why choose / why not
- Choose a key such as tenant or customer ID when: nearly every request is scoped to one tenant or
  customer, so its reads and writes stay on one shard.
- Don't shard when: the dominant queries need cross-entity joins, multi-entity transactions or
  full-dataset aggregations; fan-out and distributed coordination can outweigh the scaling gain.
- Avoid an auto-increment ID or a timestamp as the key when: new rows are the busiest rows, because
  they all land on one shard; a hash of the key spreads sequential values across shards.

## Interview angle
- Probed as "you sharded by user ID; now build last month's report of all orders"; the answer is a
  fan-out query or an externally maintained index, not a join.
- Common wrong answer: "the database handles it"; with application-level sharding, routing,
  fan-out and cross-shard consistency are the application's job.
- Strong answer: pick the key from the dominant access pattern, name the queries it makes
  expensive, and say how you would serve those.

## Related
- [[A write-heavy database scales by sharding, because replicas add only read capacity]]: that note
  says when to shard; this one is the bill, which is why that note rules out cheaper levers first.
- [[Stateless services scale out, while scaling up one machine stops at a hardware limit]]: scaling
  out a stateless tier is cheap because it holds no data; sharding is what scaling out costs once
  the data itself must be split.
- [[System design MOC]]: the scaling part of the System design cluster, for the follow-up
  question after "we sharded".
