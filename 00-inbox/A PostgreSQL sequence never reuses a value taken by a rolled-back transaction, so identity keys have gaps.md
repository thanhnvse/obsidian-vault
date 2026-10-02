---
tags: [database, sql, schema-design, keys, postgresql, interview]
status: draft
author: claude
up: ["[[Database MOC]]"]
source: "https://www.postgresql.org/docs/15/functions-sequence.html"
created: 2026-10-01
score: 0.907
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A PostgreSQL sequence never reuses a value taken by a rolled-back transaction, so identity keys have gaps

## Core idea
An identity column, `bigint GENERATED ALWAYS AS IDENTITY`, takes its values from a sequence through
`nextval`. So that concurrent transactions drawing from the same sequence never block each other,
a value handed out by `nextval` is not reclaimed when the calling transaction aborts. A rolled-back
insert or a database crash therefore leaves a gap, and so can an `INSERT ... ON CONFLICT`, which
calls `nextval` before it detects the conflict. The PostgreSQL 15 documentation concludes that
sequence objects cannot be used to obtain gapless sequences. On PostgreSQL 15.19 an identity value
taken by a rolled-back insert was never reused: the next committed insert received the following
value.

## Why choose / why not
- Use an identity key when: the column only has to identify the row, such as `orders.id`; a gap
  costs nothing, and concurrent inserts never wait for each other's commit.
- Don't use it as an invoice or receipt number when: an auditor or the law requires a gapless
  series; keep a counter row per series and increment it in the business transaction with
  `UPDATE invoice_series SET last_no = last_no + 1 ... RETURNING last_no`, accepting that every
  invoice of that series then waits for the previous one to commit.
- Don't read a gap as "a row was deleted": a rollback, a crash or an `ON CONFLICT` leaves the same
  gap.

## Interview angle
- Probed as "our order ids are sequential; can we print them as invoice numbers?".
- Common wrong answer: "a sequence has no gaps unless someone deletes rows."
- Strong answer: `nextval` is deliberately non-transactional so inserts do not serialise; a
  gapless number needs a counter updated in the same transaction, which does serialise the
  writers of that series.

## Related
- [[SELECT FOR UPDATE makes competing writers queue on the row until the lock holder commits]]: a
  gapless counter row is that same queue, because each invoice transaction holds the counter row
  lock until it commits.
