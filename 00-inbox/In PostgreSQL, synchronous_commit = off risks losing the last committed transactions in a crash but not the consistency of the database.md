---
tags: [database, postgresql, durability, wal, interview]
status: draft
author: claude
up: ["[[Database MOC]]"]
source: "https://www.postgresql.org/docs/15/wal-async-commit.html"
created: 2026-10-07
review: unjudged
---
# In PostgreSQL, synchronous_commit = off risks losing the last committed transactions in a crash but not the consistency of the database

## Core idea
PostgreSQL writes the log first: data files change only after their log records are flushed. With
the default `synchronous_commit = on`, `COMMIT` returns only after the transaction's write-ahead log
(WAL) is flushed, so it is durable (it survives a crash); data pages follow later, and a crash is
repaired by replaying the log. With `off`, the server reports success before the flush; the WAL
writer flushes within at most three times `wal_writer_delay`. A crash inside that window loses
the unflushed transactions, and the database is left as if they had been aborted cleanly. The
setting is per transaction: `SET LOCAL synchronous_commit TO OFF` affects only that transaction,
which is still atomic and immediately visible. The lab did not show a lost transaction; that part
comes from the manual.

## Why choose / why not
- Choose `off` for chosen transactions when: the write can be recreated, or nobody has been told
  about it yet, such as a view counter or a click log; you get lower commit latency with no
  corruption risk.
- Don't choose it when: the write is money, an order, or anything already announced to a user or
  another service; the failure shows only after a crash, as an acknowledged order that is missing.
- Don't confuse it with `fsync = off`: that one can corrupt the cluster after a crash. Testcontainers'
  `PostgreSQLContainer` 1.21.4 starts the server with `fsync=off`, so a durability test needs it
  switched back on.

## Interview angle
- Asked as "when would you turn `synchronous_commit` off?", or "how does a commit become durable
  without writing every changed page?".
- Common wrong answer: "`synchronous_commit = off` can corrupt the database", or "durable means the
  data is on disk when the statement returns".
- Strong answer: log first, flush the log at commit, pages later, replay after a crash; `off` is
  per transaction and risks only the last moments, not consistency; it is not `fsync = off`; and
  replication or failover is a separate question.

## Related
- [[Database MOC]]: durability is the D of the guarantees this map asks about, and `synchronous_commit`
  is how PostgreSQL lets you trade it per transaction.
- [[Read-your-writes on an asynchronous PostgreSQL replica needs recency routing, WAL-position routing or remote_apply]]:
  the same `synchronous_commit` setting with `remote_apply` makes a commit wait for a standby to
  replay it; `off` is the opposite end of that dial, where the commit waits for nothing, not even the
  local log flush.
- [[Write-behind caching makes the cache the system of record until its queue is flushed]]: the same
  trade at another layer; acknowledge first, make it durable a moment later, and accept that a crash
  in between loses the last writes.

Written up in win-interview: backend/docs/acid.md, section 2.4
