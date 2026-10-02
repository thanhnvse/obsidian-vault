---
tags: [database, sql, indexing, postgresql, interview]
status: draft
author: claude
source: "https://www.postgresql.org/docs/18/indexes-multicolumn.html"
created: 2026-09-30
score: 0.877
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A composite B-tree index is most efficient when the query constrains its leading columns

## Core idea
A composite B-tree index on `orders(customer_id, created_at)` is sorted by `customer_id` first,
and by `created_at` only within equal `customer_id` values. PostgreSQL 18 can use it for
conditions on any subset of its columns, but it is most efficient when the query constrains the
leading columns: equality on the leading columns, plus an inequality on the first column without
an equality, limits the part of the index that is scanned. So `WHERE customer_id = ? AND
created_at > ?` reads one narrow slice of the index, while `WHERE created_at > ?` alone has no
leading-column constraint to narrow the scan. Column order is therefore part of the index
design, not a detail the planner fixes later.

## Why choose / why not
- Choose a composite index when: the hot query filters on the same pair every time, such as one
  customer's orders in a date range; put the equality column (`customer_id`) first and the
  range or sort column (`created_at`) last.
- Don't add a separate index on `customer_id` when: the composite index already leads with it;
  it serves `WHERE customer_id = ?` too, and the extra index only slows writes.
- Use two single-column indexes instead when: queries filter sometimes on `customer_id`,
  sometimes on `created_at`, sometimes on both; PostgreSQL can combine them with a bitmap scan,
  while the composite index is less useful for the `created_at`-only query.

## Interview angle
- Probed as "does an index on `(customer_id, created_at)` help `WHERE created_at > ?`"; the
  expected answer is "much less, because `created_at` is not the leading column".
- Common wrong answer: "column order does not matter, the optimizer reorders it"; the optimizer
  can reorder the `WHERE` clause, not the sort order of the index.
- Strong answer: order the columns by how the query constrains them, equality first and range
  last; PostgreSQL 18 can skip-scan a trailing-column query only when the leading column has
  very few distinct values.

## Related
- [[A B-tree index speeds lookups and range scans at the cost of slower writes and more storage]]:
  a composite index is the same sorted structure keyed by a tuple, so it has the same write
  cost, and its leading-column rule follows directly from that sort order.
