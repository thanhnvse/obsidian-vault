---
tags: [database, sql, performance, pagination, postgresql, interview]
status: draft
author: claude
up: ["[[Indexes and query performance MOC]]"]
source: "https://www.postgresql.org/docs/15/functions-comparisons.html"
created: 2026-10-01
score: 0.783
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# In PostgreSQL 15 a keyset condition written as a row-value comparison is an index bound, while the equivalent OR is only a filter

## Core idea
Keyset pagination on `ORDER BY created_at, id` continues after the last row seen. It can be written
as a row-value comparison, `(created_at, id) > (:t, :id)`, or as
`created_at > :t OR (created_at = :t AND id > :id)`. Both return the same rows: the PostgreSQL
documentation defines `ROW(a, b) > ROW(c, d)` as equivalent to `a > c OR (a = c AND b > d)`,
comparing elements left to right and stopping at the first pair that differs. Since PostgreSQL
8.2 a row comparison can be used as an index condition for a multicolumn index that matches the
row value, so with a B-tree index on `(created_at, id)` the scan starts right after the last row.
PostgreSQL 15 does not turn the OR form into such a starting bound. On PostgreSQL 15.19, on page
2,001 of a feed, the row-value form read 10 rows through an `Index Cond`, while the OR form scanned
the same index from its start, applied the condition as a `Filter`, and discarded 20,000 rows to
return 10. Written with OR, keyset pagination costs more the deeper the page, like OFFSET.

## Why choose / why not
- Write the row-value form when: every column of the sort key runs in the same direction and an
  index has the same column order.
- Add a bound on the leading column when: the sort mixes directions, such as `created_at DESC,
  id ASC`, so no row value matches the order; `created_at <= :t AND (created_at < :t OR id > :id)`
  gives the index a range to start from.
- Check the plan in each engine when: the query must run on several databases; support for row
  values and their use as index bounds differs between engines.

## Interview angle
- Probed as "write the WHERE clause for the next page", or "keyset pagination is slow on deep pages
  here; why?".
- Common wrong answer: "the two forms are logically equivalent, so the planner treats them the
  same."
- Strong answer: same rows, different plan; read `Index Cond` against `Filter` and
  `Rows Removed by Filter` in the plan, and use the row value with an index in the same order.

## Related
- [[Keyset pagination stays fast on deep pages because it does not read the skipped rows]]: its
  constant cost per page only holds when the condition is an index bound, which is what this note
  shows how to write.
- [[EXPLAIN ANALYZE shows actual rows next to the planner's estimates for every plan node]]: the
  plan it prints is where `Index Cond` against `Filter` shows which form the planner received.
