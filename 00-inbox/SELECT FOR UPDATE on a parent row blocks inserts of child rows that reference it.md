---
tags: [database, concurrency, locking, postgresql, interview]
status: draft
author: claude
up: ["[[Concurrency MOC]]"]
source: "https://www.postgresql.org/docs/15/explicit-locking.html"
created: 2026-09-30
score: 0.931
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# SELECT FOR UPDATE on a parent row blocks inserts of child rows that reference it

## Core idea
In PostgreSQL 15, inserting a row with a foreign key runs a foreign-key check that takes a
`FOR KEY SHARE` lock on the referenced parent row. `FOR KEY SHARE` conflicts with `FOR UPDATE` but
not with `FOR NO KEY UPDATE`. So a batch job that locks a customer with `SELECT ... FOR UPDATE` to
change its running total makes every concurrent `INSERT` of an order for that customer wait until
the job commits. `SELECT ... FOR NO KEY UPDATE`, and any plain `UPDATE` that changes no key column,
take only the weaker lock and let the order inserts through. Both sides were reproduced on
PostgreSQL 15.19 with two JDBC connections.

## Why choose / why not
- Choose `FOR NO KEY UPDATE` when: the transaction locks a parent row only to change non-key
  columns, such as a customer's running total; child inserts keep flowing.
- Keep `FOR UPDATE` when: the transaction may delete the row or change its key, which the child
  rows depend on.
- Skip the explicit lock when: one statement such as `UPDATE customer SET total = total + ?` does
  the change; it takes the weaker lock by itself.

## Interview angle
- Probed as "order inserts stall every night while a batch job runs; the job locks customers with
  `FOR UPDATE`. Why?"
- Common wrong answer: "a row lock only affects writers of the locked table."
- Strong answer: name the foreign-key check's `FOR KEY SHARE` lock, the row-lock conflict table,
  and `FOR NO KEY UPDATE` as the fix.

## Related
- [[SELECT FOR UPDATE makes competing writers queue on the row until the lock holder commits]]:
  that note covers writers of the same row; this one shows the same lock also queues inserts into
  other tables that reference the row.
