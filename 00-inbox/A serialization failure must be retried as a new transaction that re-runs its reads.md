---
tags: [database, isolation, transactions, postgresql, interview]
status: draft
author: claude
up: ["[[Database MOC]]"]
source: "https://www.postgresql.org/docs/15/mvcc-serialization-failure-handling.html"
created: 2026-10-01
score: 0.885
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 2
---
# A serialization failure must be retried as a new transaction that re-runs its reads

## Core idea
In PostgreSQL 15, a serialization failure (SQLSTATE `40001`) is an error, and an error puts the
transaction in an aborted state: every later statement in it fails with `25P02`
(`in_failed_sql_transaction`) until the client rolls it back. So the only retry that can work
starts a new transaction. The manual says to retry the complete transaction, "including all logic
that decides which SQL to issue and/or which values to use", and explains that PostgreSQL offers
no automatic retry because it cannot do one with any guarantee of correctness. Re-running the
reads is what makes the retry correct: the new transaction sees the other transaction's commit
and can decide differently, while replaying the old `UPDATE` statements with values computed from
the old reads would recreate the anomaly.

## Why choose / why not
- Wrap the retry around the code that starts the transaction when: it runs at REPEATABLE READ or
  SERIALIZABLE, or can meet a deadlock (`40P01`, which the manual also suggests retrying); in
  Spring that is outside the outermost `@Transactional` method, with a cap on attempts and a
  backoff, because the manual warns that a retry is not guaranteed to succeed.
- Don't put side effects inside the retried block: an e-mail or an HTTP call made by attempt 1 is
  not rolled back by the retry; send it after the commit, or from an outbox.
- Don't retry a single statement after catching the exception: the transaction is already
  aborted, so the next statement fails with `25P02`.

## Interview angle
- Probed as "you raised the isolation level and now see `could not serialize access`; what do you
  do?"
- Common wrong answer: "catch the exception and run the failed statement again."
- Strong answer: roll back, start a new transaction, re-run every read and decision, bound the
  attempts with backoff, keep side effects out, and retry only `40001` and `40P01`.

## Related
- [[A deadlock aborts the whole transaction in PostgreSQL but only one statement in Oracle]]: the
  deadlock victim is left in the same aborted state, so this retry rule applies to `40P01` too.
- [[Catching an exception from a joined REQUIRED method ends in UnexpectedRollbackException]]: a
  retry inside a joined Spring transaction fails for a similar reason, since the shared
  transaction is already doomed when the inner call throws.
- [[PostgreSQL SERIALIZABLE can abort transactions that changed different rows when their reads scanned the whole table]]:
  those false-positive aborts are survivable only because of this retry.
