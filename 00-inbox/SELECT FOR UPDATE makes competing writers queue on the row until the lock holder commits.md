---
tags: [database, concurrency, locking, postgresql, interview]
status: draft
author: claude
source: "https://www.postgresql.org/docs/18/explicit-locking.html"
created: 2026-09-30
score: 0.87
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# SELECT FOR UPDATE makes competing writers queue on the row until the lock holder commits

## Core idea
In a flash sale, every buyer's transaction runs `SELECT stock FROM product WHERE id = ? FOR
UPDATE`, checks the stock, decrements it and commits. In PostgreSQL 18, `FOR UPDATE` locks the
returned row as though for update, so every other transaction that tries to update or lock that
row waits until the lock holder commits or rolls back. Under READ COMMITTED the next buyer in
the queue then gets the newly committed row and decides on the current stock, instead of
failing and retrying as it would with a version column. The lock lasts until the transaction
ends, so the queue moves only as fast as each transaction is short.

## Why choose / why not
- Choose `FOR UPDATE` when: many writers hit the same row and each transaction is short, such as
  decrementing stock during a flash sale; competitors queue instead of failing and retrying.
- Don't hold the lock across slow work: a transaction that waits on user input or an outbound
  call keeps every buyer waiting; use a version column for that flow instead.
- Lock rows in one consistent order, such as by primary key, when a transaction locks several
  stock rows; PostgreSQL detects a deadlock and aborts one transaction, but the order avoids it.

## Interview angle
- Probed as "optimistic or pessimistic locking, and why?"; the interviewer wants contention and
  transaction length as the deciding factors.
- Common wrong answer: "`FOR UPDATE` blocks readers"; in PostgreSQL a plain `SELECT` still reads
  the last committed version of the row.
- Strong answer: pessimistic for short and hot, optimistic for long and rarely contended; then
  add a consistent lock order and a bounded wait with `NOWAIT` or `lock_timeout`.

## Related
- [[A version column detects a lost update at write time instead of blocking the other writer]]:
  the two are alternatives for the same lost update, and contention and transaction length
  decide between them.
- [[Write skew survives snapshot isolation because the two transactions write different rows]]:
  locking the rows a rule reads with `FOR UPDATE` is one of that note's fixes that does not need
  SERIALIZABLE.
