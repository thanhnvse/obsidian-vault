---
tags: [database, concurrency, locking, postgresql, oracle, interview]
status: draft
author: claude
up: ["[[Concurrency MOC]]"]
source: "https://www.postgresql.org/docs/15/explicit-locking.html"
created: 2026-10-01
score: 0.85
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 2
---
# A deadlock aborts the whole transaction in PostgreSQL but only one statement in Oracle

## Core idea
Two transactions that lock the same two rows in opposite order wait for each other, so the database has to break the cycle. PostgreSQL detects the deadlock and resolves it by aborting one of the transactions involved, and which one is aborted is difficult to predict. The victim gets SQLSTATE 40P01; its earlier changes are rolled back, and every later statement in it fails with 25P02 until the client issues ROLLBACK. Oracle Database instead rolls back only the statement that detected the deadlock and returns ORA-00060; the transaction keeps its earlier changes and locks, and the documentation says it should usually be rolled back explicitly.

## Why choose / why not
- Retry the whole transaction on 40P01 when: running on PostgreSQL; the work is already gone, so a statement-level retry is impossible.
- Roll back explicitly after ORA-00060 when: running on Oracle; otherwise the transaction keeps half its work and its locks.

## Interview angle
- Probed as "what happens when two transactions deadlock?"
- Common wrong answers: "both hang until a timeout" or "the younger one is always killed."
- Strong answer: detection, one victim, then exactly what each database rolls back and what the client must do next.

## Related
- [[SELECT FOR UPDATE makes competing writers queue on the row until the lock holder commits]]: a deadlock is two of those queues waiting on each other in opposite order.
- [[SELECT FOR UPDATE on a parent row blocks inserts of child rows that reference it]]: a less obvious lock that can close the same kind of cycle.
