---
tags: [database, concurrency, race-condition, postgresql, interview]
status: draft
author: claude
up: ["[[Concurrency MOC]]"]
source: "https://www.postgresql.org/docs/15/transaction-iso.html"
created: 2026-10-01
score: 0.91
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A conditional UPDATE with the stock check in its WHERE clause cannot oversell under READ COMMITTED

## Core idea
`UPDATE item SET stock = stock - 1 WHERE id = :id AND stock >= 1` does the check and the decrement
in one statement. When two application instances sell the last item at the same time under
PostgreSQL 15's READ COMMITTED, the second `UPDATE` waits for the first one's row lock. After the
first transaction commits, PostgreSQL re-evaluates the second `UPDATE`'s `WHERE` clause against the
newest row version; stock 0 no longer matches `stock >= 1`, so the second `UPDATE` changes 0 rows
and the stock stops at 0. A check done by a separate `SELECT` in the application, followed by an
atomic `SET stock = stock - 1`, does not get this protection: both instances read stock 1, both
decrements apply, and the stock ends at -1. The row count is the answer to the check, so the
application must turn 0 rows into "sold out".

## Why choose / why not
- Choose a conditional `UPDATE` when: the rule is about the row being changed and fits in its
  `WHERE` clause, such as `stock >= :qty` or `status = 'pending'`; it needs no lock held across
  application code, no version column and no retry loop.
- Don't choose it when: the decision needs application logic or other rows; use a version column,
  `SELECT ... FOR UPDATE`, or SERIALIZABLE with a retry.
- Don't expect the same outcome at REPEATABLE READ: there the waiting `UPDATE` fails with SQLSTATE
  `40001` after the first commit instead of changing 0 rows, so the caller needs a retry.

## Interview angle
- Probed as "several instances sell a limited item; how do you stop overselling the last one?"
- Common wrong answer: "check the stock with a `SELECT`, then run an atomic decrement", or "wrap it
  in `@Transactional`."
- Strong answer: one conditional `UPDATE`; explain that the waiting update re-checks its `WHERE`
  clause on the newest row version; and say that the code reads the row count and maps 0 to "sold
  out".

## Related
- [[A synchronized block cannot prevent a lost update between two application instances]]: that
  note moves the guard into the database; this note is the one-statement form of that guard when
  the rule includes a check.
- [[A version column detects a lost update at write time instead of blocking the other writer]]: a
  version check is the same conditional `UPDATE` with the version as the condition, needed when the
  new value is decided in application code.
- [[PostgreSQL REPEATABLE READ rejects a lost update with SQLSTATE 40001 instead of overwriting the newer row]]:
  the same race at REPEATABLE READ ends in an error instead of 0 rows.
