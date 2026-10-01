---
tags: [database, sql, indexing, postgresql, interview]
status: draft
author: claude
up: ["[[Indexes and query performance MOC]]"]
source: "https://www.postgresql.org/docs/15/indexes-partial.html"
created: 2026-10-01
score: 0.912
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A partial index cannot serve a generic plan whose bind parameter decides the predicate

## Core idea
A partial index holds entries only for the rows that satisfy its predicate, such as
`CREATE INDEX ON orders (created_at) WHERE status = 'PENDING'`. PostgreSQL decides whether a
query can use it at planning time, by checking that the query's condition implies the predicate.
A generic plan for `WHERE status = $1` must be valid for every value of `$1`, so it cannot use an
index that only holds `PENDING` rows; a custom plan, made with the actual value `'PENDING'`, can.
In PostgreSQL 15 a prepared statement gets custom plans for its first five executions and then
switches to the generic plan when its estimated cost is not much higher, and `plan_cache_mode`
can force either. On PostgreSQL 15.19 the custom plan used the partial index and the forced
generic plan read the whole table.

## Why choose / why not
- Choose a partial index when: queries look for a small, fixed subset of rows, such as a work
  queue of pending orders; it is a fraction of the size of a full index and rows outside the
  subset never write to it.
- Write the predicate as a literal in the SQL when: a query relies on the partial index; bind
  only the other values, such as the customer id.
- Don't use a partial index when: the filtered value changes from call to call; index the column
  itself, leading a composite index if needed.

## Interview angle
- Probed as "the query uses the index in psql but not from the application"; the difference is a
  literal value against a bind parameter in a prepared statement.
- Common wrong answer: "a prepared statement is planned again with the parameter values on every
  execution."
- Strong answer: predicate matching happens when the plan is made, so name custom and generic
  plans and put the partial index's condition in the SQL text.

## Related
- [[A B-tree index speeds lookups and range scans at the cost of slower writes and more storage]]:
  a partial index is that B-tree restricted to a subset of rows, which is why it costs less on
  writes and on storage.
- [[EXPLAIN ANALYZE shows actual rows next to the planner's estimates for every plan node]]: to see
  this problem, explain the `EXECUTE` of the prepared statement, not the query with a literal.
