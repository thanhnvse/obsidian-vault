---
tags: [database, concurrency, locking, postgresql, interview]
status: draft
author: claude
up: ["[[Concurrency MOC]]"]
source: "https://www.postgresql.org/docs/15/runtime-config-client.html"
created: 2026-10-01
score: 0.887
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A PostgreSQL row-lock wait never times out by default, while NOWAIT and lock_timeout make it fail with 55P03

## Core idea
In PostgreSQL 15, a `SELECT ... FOR UPDATE` or an `UPDATE` that finds its row locked by another
transaction waits until that transaction ends, and `lock_timeout` defaults to zero, which disables
any limit on that wait. `SELECT ... FOR UPDATE NOWAIT` refuses to wait: it reports an error at once
if a row cannot be locked immediately, with SQLSTATE `55P03` (`lock_not_available`).
`SET LOCAL lock_timeout = '100ms'` lets a statement wait up to that limit and then aborts it with
`55P03`. The limit applies separately to each lock acquisition attempt and covers both explicit
locks and implicitly acquired ones, so it also bounds a plain `UPDATE` waiting for a row. If
`statement_timeout` is nonzero, a `lock_timeout` of the same or a larger value is pointless,
because the statement timeout fires first. Like any error in PostgreSQL, the failure leaves the
transaction aborted, so the client rolls back and then retries or reports "busy". The manual
advises against setting `lock_timeout` in `postgresql.conf`, because that would affect all
sessions.

## Why choose / why not
- Use `NOWAIT` when: the caller should get "busy, try again" instead of waiting at all, such as a
  user reserving a record another request is already changing.
- Use `SET LOCAL lock_timeout` when: a short wait is fine but a pile-up is not, such as an endpoint
  behind a small connection pool; it bounds every lock the transaction waits for, not one query.
- Keep the default wait when: every lock holder is a short transaction; waiting a few
  milliseconds is cheaper than a failed request and a retry.
- Use `SKIP LOCKED` instead when: any free row will do, such as workers claiming jobs.

## Interview angle
- Probed as "requests hang behind one long transaction that holds a row lock; how do you stop them
  piling up?"
- Common wrong answer: "the database times lock waits out by itself", or "catch the timeout and
  carry on in the same transaction."
- Strong answer: the default wait is unbounded; `NOWAIT` fails at once, `lock_timeout` after a limit
  per lock attempt, both with `55P03`; roll back, then retry or report busy; and keep lock holders
  short.

## Related
- [[SELECT FOR UPDATE makes competing writers queue on the row until the lock holder commits]]: that
  note's queue has no length or time limit; this note is how to bound it.
- [[SKIP LOCKED lets several workers claim different rows of a job table without waiting]]: the
  third way not to wait, which takes another row instead of failing.
- [[A serialization failure must be retried as a new transaction that re-runs its reads]]: `55P03`
  leaves the transaction aborted in the same way, so the same whole-transaction retry applies.
