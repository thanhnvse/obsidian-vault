---
tags: [database, sql, transactions, isolation, postgresql, interview]
status: draft
author: claude
source: "https://www.postgresql.org/docs/18/transaction-iso.html"
created: 2026-09-30
score: 0.847
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# READ COMMITTED gives each statement its own snapshot, so two reads in one transaction can disagree

## Core idea
In PostgreSQL 18, a query under READ COMMITTED sees a snapshot of the data committed before that
query began, not before the transaction began. So two identical
`SELECT balance FROM accounts WHERE id = 1` statements in one transaction can return different
values if another transaction commits a change in between: a non-repeatable read. A repeated
range query can likewise return rows that were not there the first time: a phantom read. The
level still rules out dirty reads, because no snapshot ever contains uncommitted data. READ
COMMITTED is the default level in PostgreSQL 18 and in Oracle Database, so this is what a
transaction gets unless it asks for more.

## Why choose / why not
- Keep READ COMMITTED when: each statement stands on its own, such as a single-row insert or
  update; PostgreSQL never fails a READ COMMITTED transaction with a serialization error, so
  there is nothing to retry.
- Raise the level to REPEATABLE READ when: one transaction must see the same balances across
  several queries, such as a report that sums accounts in two steps; PostgreSQL then gives the
  whole transaction one snapshot, and the caller must retry on serialization failure.
- Take a row lock instead when: the transaction reads a balance and then writes based on it;
  a later statement may see a newer value, so lock the row with `SELECT ... FOR UPDATE` first.

## Interview angle
- Probed as "what isolation level does your service run at, and what can a transaction see?";
  the interviewer wants the default and one anomaly it allows.
- Common wrong answer: "inside one transaction, reading the same row twice always gives the
  same value."
- Strong answer: show the two-`SELECT` example, name the default for your database (READ
  COMMITTED for PostgreSQL and Oracle, REPEATABLE READ for MySQL 8.4 InnoDB), and say which
  transactions need a stronger level or a lock.

## Related
- [[A joined REQUIRED transaction ignores its own isolation, timeout and readOnly attributes]]:
  a Spring method that joins an existing transaction cannot raise its isolation, so it runs at
  the per-statement snapshots this note describes unless the outer method set a stronger level.
