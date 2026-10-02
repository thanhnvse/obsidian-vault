---
tags: [database, sql, indexing, postgresql, interview]
status: draft
author: claude
source: "https://www.postgresql.org/docs/18/indexes-types.html"
created: 2026-09-30
score: 0.89
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A B-tree index speeds lookups and range scans at the cost of slower writes and more storage

## Core idea
A B-tree index keeps the indexed values in sorted order, so PostgreSQL can find the rows for an
equality or range condition (`=`, `<`, `>`, `BETWEEN`, `IN`) without reading the whole table,
and can return rows already sorted for an `ORDER BY`. In PostgreSQL 18, `CREATE INDEX` builds a
B-tree unless another type is named. The index is a second structure that the database must keep
synchronized with the table, so data-changing statements pay extra work to maintain it, and it
takes storage of its own. On `orders(customer_id)`, that buys a fast "orders of one customer"
lookup at the price of every order insert also writing the index.

## Why choose / why not
- Add a B-tree index when: a frequent query filters or sorts on a selective column, such as the
  orders of one customer among millions; EXPLAIN on the real query should show the scan it
  replaces.
- Don't add one when: the condition matches a large share of the table; PostgreSQL then reads
  the table sequentially anyway, so let that scan run and save the write cost.
- Drop an index when: no query uses it; every index still costs each insert, which is why the
  PostgreSQL documentation says seldom-used indexes should be removed.

## Interview angle
- Probed as "why not index every column?"; the answer is the write cost and storage each index
  adds to every insert.
- Common wrong answer: "an index always makes a query faster"; for an unselective condition the
  planner ignores it, and the table still pays for it on each write.
- Strong answer: name the query the index serves, its selectivity and its write cost, then show
  with EXPLAIN that the planner uses it.

## Related
- [[Cardinality decides where a relationship's foreign key goes]]: PostgreSQL does not index the
  foreign key column on the many side, so `orders(customer_id)` is the index most often added by
  hand.
