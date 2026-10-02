---
tags: [system-design, interview, performance, hibernate, jdbc]
status: draft
author: claude
source: "https://learn.microsoft.com/en-us/azure/architecture/antipatterns/chatty-io/"
created: 2026-09-30
score: 0.844
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Batching writes raises throughput by paying the per-request overhead once per batch

## Core idea
Each I/O request to a database, a service or a file carries significant overhead, and the
cumulative cost of many small requests slows the whole system down. Packaging the data into fewer,
larger requests pays that overhead once per batch instead of once per record. In Hibernate ORM 6.6,
JDBC batching is not enabled by default, so every insert statement costs its own database round trip
until `hibernate.jdbc.batch_size` is set. Buffering writes in memory to build a batch has a price:
the buffered data is lost if the process crashes before the batch is written.

## Why choose / why not
- Choose batching when: a write-heavy path inserts or updates many rows in one go, such as an import,
  event ingestion or a nightly job; set `hibernate.jdbc.batch_size` (the Hibernate 6.6 guide suggests
  10 to 50) and flush and clear the session regularly.
- Don't buffer in memory when: losing the pending records in a crash is unacceptable and the data
  arrives in bursts or sparsely; buffer them in an external durable queue instead.
- Keep each batch short when: the write holds locks, because a long-running write raises contention
  for the rows it locks.

## Interview angle
- Probed as "importing a million rows through JPA takes hours"; check first whether JDBC batching is
  on at all.
- Common wrong answer: "setting `hibernate.jdbc.batch_size` is enough"; Hibernate 6.6 silently
  disables insert batching for entities that use an identity identifier generator.
- Strong answer: name the round-trip cost, the batching switch, the IDENTITY trap (move the id
  generator off IDENTITY), and the durability cost of buffering.

## Related
- [[A write-heavy database scales by sharding, because replicas add only read capacity]]: batching
  is one of the cheaper write levers to try before sharding, because it raises throughput on the
  existing primary.
- [[System design MOC]]: belongs to the System design cluster's write-heavy branch, as the
  lever a Java developer controls directly from code.
