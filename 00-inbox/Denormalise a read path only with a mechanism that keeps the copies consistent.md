---
tags: [database, sql, schema-design, denormalization, postgresql, interview]
status: draft
author: claude
source: "https://www.postgresql.org/docs/18/trigger-definition.html"
created: 2026-09-30
score: 0.877
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Denormalise a read path only with a mechanism that keeps the copies consistent

## Core idea
Denormalising stores a copy of data, such as an `orders.total` column that repeats the sum of
the order's lines, so a hot read skips the join and the aggregate. The copy brings back the
update anomaly that third normal form removes: every write to an order line must also change the
total, or the total is silently wrong. A denormalised copy is therefore only safe when one
mechanism owns keeping it in step with its source. In PostgreSQL 18, a trigger on the lines
table can be that mechanism, because a trigger runs in the same transaction as the statement
that fired it and commits or rolls back with it.

## Why choose / why not
- Denormalise when: a measured hot read, such as an order list that shows totals, is dominated
  by the join or aggregate and an index does not fix it.
- Let a trigger own the total when: readers must never see a total that disagrees with the
  lines; the price is slower line writes and contention on the order row.
- Accept a stale copy instead when: the read tolerates totals as of a refresh, such as a daily
  report; a materialized view then owns the copy, refreshed with `REFRESH MATERIALIZED VIEW`.
- Don't denormalise when: no single mechanism owns the total; application code that updates it
  in one write path and forgets another produces wrong totals with no error.

## Interview angle
- Probed as "when would you denormalise?"; the interviewer checks whether you name the mechanism
  that keeps the copy correct, not only the speed-up.
- Common wrong answer: "denormalise for performance", with nothing on which writes must update
  the copy.
- Strong answer: name the measured read path, the copy, and the one mechanism that keeps it in
  step, plus how stale a reader may see it.

## Related
- [[Third normal form removes update anomalies by making every non-key column depend only on the key]]:
  denormalising deliberately breaks that rule, so the update anomaly it describes is exactly the
  risk the owning mechanism has to close.
