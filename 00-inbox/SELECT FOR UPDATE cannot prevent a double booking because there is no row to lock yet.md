---
tags: [database, concurrency, race-condition, postgresql, interview]
status: draft
author: claude
up: ["[[Concurrency MOC]]"]
source: "https://www.postgresql.org/docs/15/index-unique-checks.html"
created: 2026-10-01
score: 0.87
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# SELECT FOR UPDATE cannot prevent a double booking because there is no row to lock yet

## Core idea
A check-then-insert such as "if seat 7A has no booking, insert one" runs a `SELECT` and then an
`INSERT`. Adding `FOR UPDATE` to the `SELECT` does not help: while the seat is free the `SELECT`
returns no row, so nothing is locked, and a second instance runs the same check without waiting;
both insert, and the seat is sold twice. A row lock needs a row; what the two inserts share is
only a key. In PostgreSQL 15 a unique constraint on that key is the guard that sees the other
insert while it is still in flight: if a conflicting row was inserted by a transaction that is
still open, the second inserter waits to see whether that transaction commits. If it commits, the
second insert fails with SQLSTATE `23505` (`unique_violation`); if it rolls back, there is no
conflict.

## Why choose / why not
- Choose a unique constraint when: the rule is uniqueness, such as one booking per seat, one
  account per e-mail address or one payment per idempotency key; the database enforces it for every
  writer, including scripts.
- Use `ON CONFLICT ... DO NOTHING` when: the loser should hear "already taken" and keep working in
  the same transaction; catching `23505` instead leaves the transaction aborted.
- Use a transaction-level advisory lock on a key, or SERIALIZABLE with a retry, when: the rule is
  not uniqueness but still concerns rows that do not exist yet, such as at most three bookings per
  customer per day.

## Interview angle
- Probed as "how do you prevent a double booking when the service runs on several instances?"
- Common wrong answer: "lock the seat with `SELECT ... FOR UPDATE` first."
- Strong answer: there is no row to lock; a unique constraint makes the second inserter wait for the
  uncommitted first insert and then fail with `23505`; `ON CONFLICT DO NOTHING` turns that into 0
  rows; an advisory lock covers rules that are not uniqueness.

## Related
- [[SELECT FOR UPDATE makes competing writers queue on the row until the lock holder commits]]: that
  queue forms on a row that exists; with no row there is no queue, which is why the lock fails here.
- [[Write skew survives snapshot isolation because the two transactions write different rows]]:
  that note warns that row locks cannot protect rows that do not exist yet; this note gives the fix
  when the rule is uniqueness.
- [[A synchronized block cannot prevent a lost update between two application instances]]: the same
  several-instance setting, where the guard has to live in the database.
