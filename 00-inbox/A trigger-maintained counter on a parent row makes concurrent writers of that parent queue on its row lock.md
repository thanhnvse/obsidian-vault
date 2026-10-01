---
tags: [database, sql, denormalization, locking, postgresql, interview]
status: draft
author: claude
up: ["[[Database MOC]]"]
source: "https://www.postgresql.org/docs/15/explicit-locking.html"
created: 2026-10-01
score: 0.913
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A trigger-maintained counter on a parent row makes concurrent writers of that parent queue on its row lock

## Core idea
A denormalised counter such as `customer.order_count` is often kept by a trigger on `orders` that
runs `UPDATE customer SET order_count = order_count + 1 WHERE id = NEW.customer_id`. The trigger
runs inside the inserting transaction, so the count commits or rolls back with the order. That
`UPDATE` also takes a row-level lock on the customer row, and PostgreSQL holds row-level locks
until the transaction ends. A second transaction inserting an order for the same customer must
update the same row, so it waits until the first one commits or rolls back. Inserts that were
independent become a queue per parent row. On PostgreSQL 15.19 the second writer waited on the
customer row, and with `lock_timeout` set it failed with SQLSTATE `55P03`.

## Why choose / why not
- Choose a trigger counter when: the count is read far more often than it changes, and writes for
  the same parent rarely overlap, such as orders per customer in a shop where each customer orders
  a few times a month.
- Don't use one when: many concurrent writes hit the same parent, such as likes on a viral post;
  count from the child table with an index on its foreign key, or insert rows into a delta table
  and fold them into the counter periodically.
- Keep the inserting transactions short when: the counter must stay; the lock is held until
  commit, so any slow work after the insert lengthens the queue.

## Interview angle
- Probed as "you keep `order_count` in sync with a trigger; what does that cost?".
- Common wrong answer: "a trigger keeps it consistent, so there is no cost."
- Strong answer: same-transaction consistency, paid for with a row lock on the parent that
  serialises its writers until commit; `lock_timeout` turns the wait into `55P03`; then name the
  alternatives for a hot parent.

## Related
- [[Denormalise a read path only with a mechanism that keeps the copies consistent]]: it picks a
  trigger as the mechanism that owns the copy and mentions contention, and this note explains
  where that contention comes from.
- [[SELECT FOR UPDATE makes competing writers queue on the row until the lock holder commits]]: the
  trigger's `UPDATE` takes a row lock implicitly, so writers of one parent queue exactly as they do
  behind an explicit `FOR UPDATE`.
- [[A PostgreSQL row-lock wait never times out by default, while NOWAIT and lock_timeout make it fail with 55P03]]:
  the queue behind the counter row has no time limit by default, and that note shows how
  `lock_timeout` bounds it.
