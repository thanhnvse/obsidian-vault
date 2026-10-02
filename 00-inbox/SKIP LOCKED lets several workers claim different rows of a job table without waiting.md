---
tags: [database, concurrency, locking, postgresql, interview]
status: draft
author: claude
up: ["[[Concurrency MOC]]"]
source: "https://www.postgresql.org/docs/15/sql-select.html"
created: 2026-09-30
score: 0.867
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# SKIP LOCKED lets several workers claim different rows of a job table without waiting

## Core idea
Several workers poll one job table, and each runs
`SELECT id FROM job WHERE status = 'queued' ORDER BY id LIMIT 1 FOR UPDATE SKIP LOCKED`, processes
the job and commits. In PostgreSQL 15, `SKIP LOCKED` skips any selected row that cannot be locked
immediately, so the second worker gets job 2 at once instead of waiting behind the worker that
holds job 1. With a plain `FOR UPDATE`, every worker would queue on the same first row. The manual
warns that skipping locked rows gives an inconsistent view of the data, so the option suits
queue-like tables, not general queries; MySQL 8.4 InnoDB has the same option with the same warning.

## Why choose / why not
- Choose `SKIP LOCKED` when: several consumers drain a queue-like table in the same database as
  the work, such as a job table or an outbox relay, and any free row will do.
- Don't choose it when: the query must see every matching row, such as a report or a balance
  check; locked rows silently drop out of the result.
- Use a message broker instead when: the work needs fan-out to several consumer groups, or
  ordering per key across many consumers.

## Interview angle
- Probed as "several instances poll the same jobs table; how do you stop them processing the same
  job, or blocking each other?"
- Common wrong answer: "`FOR UPDATE` is enough"; it prevents double processing but makes the
  workers queue on one row.
- Strong answer: `FOR UPDATE SKIP LOCKED` with a `LIMIT`, the status change in the same
  transaction, and the inconsistent-view caveat.

## Related
- [[SELECT FOR UPDATE makes competing writers queue on the row until the lock holder commits]]:
  `SKIP LOCKED` is the variant for the case where that queueing is exactly what you do not want.
- [[A queue in front of a service levels load spikes at the cost of an immediate response]]: a job
  table claimed with `SKIP LOCKED` is such a queue built inside the database, with the same
  trade-off of later completion for surviving the peak.
