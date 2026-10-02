---
tags: [system-design, interview, caching, consistency, durability]
status: draft
author: claude
source: "https://docs.oracle.com/en/middleware/fusion-middleware/coherence/14.1.2/develop-applications/caching-data-sources.html"
created: 2026-09-30
score: 0.877
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Write-behind caching makes the cache the system of record until its queue is flushed

## Core idea
With write-through caching, an update to the cache does not return until the cache has stored the
data in the underlying data source, so every write still pays the data source's latency. With
write-behind caching, as documented for Oracle Coherence 14.1.2, modified entries go onto a
write-behind queue and are written to the data source asynchronously after a configured delay;
several changes to the same entry within that interval are coalesced into one write. The
application no longer waits for the database, and database load drops. The price is that the cache
is the system of record until the queue has been written: the cache transaction completes before
the database transaction begins, so a later database failure needs a rollback strategy, and updates
may reach the database out of order.

## Why choose / why not
- Choose write-through when: every acknowledged write must already be in the database, such as an
  order or a payment; accept the data source's write latency on every update.
- Choose write-behind when: the same entries are updated many times in a short interval and write
  latency matters, and the business accepts data that is held by the cache cluster, not yet on disk.
- Don't choose write-behind when: other applications write the same tables, because nothing
  guarantees a queued update will not conflict with their changes.

## Interview angle
- Probed as "what happens to a write if the cache dies before the flush?"; it depends on whether the
  pending writes are replicated across the cache cluster or held by one process.
- Common wrong answer: "write-behind is a faster write-through"; it moves where the truth lives.
- Strong answer: name the system-of-record shift, the coalescing gain, and what it demands of the
  database side: tolerate failed and reordered writes, and have no other writers.

## Related
- [[Cache-aside loads data on a miss and leaves cache consistency to the application]]: cache-aside
  keeps the database as the truth and the cache as a disposable copy; write-behind inverts that
  until the flush.
- [[Batching writes raises throughput by paying the per-request overhead once per batch]]:
  write-behind is batching done by the cache, with the same cost that pending writes exist outside
  the database for a while.
- [[System design MOC]]: the caching part of the System design cluster, for the question
  "which write strategy does your cache use?".
