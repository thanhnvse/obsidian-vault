---
tags: [database, isolation, transactions, postgresql, interview]
status: draft
author: claude
up: ["[[Database MOC]]"]
source: "https://www.postgresql.org/docs/15/transaction-iso.html"
created: 2026-09-30
score: 0.891
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# PostgreSQL SERIALIZABLE can abort transactions that changed different rows when their reads scanned the whole table

## Core idea
PostgreSQL 15 SERIALIZABLE records what each transaction read as predicate locks, and aborts one
transaction with SQLSTATE `40001` when the read/write dependencies between concurrent transactions
fit no serial order. The locks follow the data actually read, and "a sequential scan will always
necessitate a relation-level predicate lock". So when two transactions each count the doctors of
their own shift through a sequential scan and each takes one doctor of their own shift off call,
the second commit fails even though no rule was at risk. This was reproduced on PostgreSQL 15.19.
The detection is conservative: it never lets an anomaly through, but it can abort work that would
have been safe.

## Why choose / why not
- Choose SERIALIZABLE when: rules span rows and every transaction can be retried; false positives
  then cost only extra attempts.
- Don't expect a low abort rate when: the hot transactions read through sequential scans or wide
  ranges; add indexes that turn the reads into index scans, keep transactions short, and declare
  readers `READ ONLY`.
- Stay at READ COMMITTED with a targeted lock or constraint when: a failed attempt is expensive to
  repeat, such as a long batch step.

## Interview angle
- Probed as "SERIALIZABLE does not block in PostgreSQL, so why not use it everywhere?"
- Common wrong answer: "it only aborts transactions that really conflict."
- Strong answer: predicate locks follow what was read, a sequential scan locks the whole table,
  so aborts include false positives; every transaction needs a retry, and indexes reduce the rate.

## Related
- [[Write skew survives snapshot isolation because the two transactions write different rows]]:
  SERIALIZABLE is that note's fix for write skew; this note is the price of that fix.
