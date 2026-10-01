---
tags: [database, sql, indexing, postgresql, interview]
status: draft
author: claude
up: ["[[Indexes and query performance MOC]]"]
source: "https://www.postgresql.org/docs/15/indexes-expressional.html"
created: 2026-10-01
score: 0.907
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Wrapping an indexed column in a function hides it from a plain PostgreSQL index

## Core idea
An index on `users(email)` stores the values of `email` in sorted order. A condition such as
`WHERE lower(email) = 'user4242@example.com'` asks about a different value, `lower(email)`, which
that index does not store, so the planner cannot use the index for it and reads the table. The
same applies to a cast or arithmetic on the column, such as `WHERE amount + 0 > 10`. There are two
fixes. Rewrite the condition on the bare column when the function only expresses a range, for
example a day as `created_at >= :day AND created_at < :day + interval '1 day'`. Or create an
expression index on the same expression, `CREATE INDEX ON users (lower(email))`, and write the
query with exactly that expression. An expression index computes its value on every insert and
non-HOT update, not during the search. On PostgreSQL 15.19 `lower(email) = ...` was a sequential
scan with an index on `email`, and an index lookup once `lower(email)` was indexed.

## Why choose / why not
- Rewrite on the bare column when: the function only turns the column into a range or a unit,
  such as one calendar day of a timestamp; the existing index serves it, and nothing new is
  written on each insert.
- Add an expression index when: the function result is the real lookup key, such as a
  case-insensitive email login; a `UNIQUE` expression index on `lower(email)` also stops two
  accounts that differ only in case.
- Don't add an expression index when: the table is write-heavy and the lookup is rare; the
  expression is computed on every insert and non-HOT update.

## Interview angle
- Probed as "why is my index not used?", with a query that applies `lower()` or a cast to the
  indexed column.
- Common wrong answer: "the index is on `email`, so any condition on `email` uses it."
- Strong answer: an index only matches what it stores; rewrite the condition on the bare column
  or index the expression, then confirm in the plan that the `Filter` on the expression became an
  `Index Cond`.

## Related
- [[A B-tree index speeds lookups and range scans at the cost of slower writes and more storage]]:
  an expression index is that B-tree built over a computed value, so it has the same write cost
  plus the computation on each write.
- [[EXPLAIN ANALYZE shows actual rows next to the planner's estimates for every plan node]]: the
  plan it prints shows the symptom, a sequential scan whose `Filter` wraps the column in the
  function.
