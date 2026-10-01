---
tags: [database, sql, performance, postgresql, interview]
status: draft
author: claude
source: "https://www.postgresql.org/docs/18/using-explain.html"
created: 2026-09-30
score: 0.863
review: "borderline"
score_reasons: ["atomic: 0.56 (borderline)", "numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 2
---
# EXPLAIN ANALYZE shows actual rows next to the planner's estimates for every plan node

## Core idea
Plain `EXPLAIN` prints the plan the PostgreSQL planner chose, with estimated rows per node,
without running the query. `EXPLAIN ANALYZE` actually executes the query and prints, next to
each node's estimate, the rows that node really produced. That comparison is the point: a node
estimated at 10 rows that returns 100 000 means every plan choice above it was made on a wrong
number. Row estimates come from the table statistics that `ANALYZE` collects, so such a gap
often points to missing or out-of-date statistics. For a node that runs in a loop, the actual
rows are a per-loop average, so multiply by `loops` before comparing.

## Why choose / why not
- Use `EXPLAIN ANALYZE` when: a query is slow and you need to find the node whose estimate went
  wrong; read the plan from the innermost nodes outward and stop at the first large gap.
- Use plain `EXPLAIN` instead when: the statement is too slow or too risky to execute, such as
  a large `DELETE` in production; it shows the estimates without running anything.
- Don't compare against a plan from a small test database when: production has other data
  volumes; the estimates follow each database's own statistics.

## Interview angle
- Probed as "a query is slow, what do you do first?"; the expected answer is a real plan with
  actual rows, not an index added by guess.
- Common wrong answer: reading the top-line cost as milliseconds; cost is in the planner's
  arbitrary units, and only the "actual" figures are measured.
- Strong answer: find the lowest node where estimated and actual rows diverge, explain why
  (stale statistics, correlated columns), and fix that before touching indexes.

## Related
- [[A B-tree index speeds lookups and range scans at the cost of slower writes and more storage]]:
  the planner chooses an index only when its row estimate says the condition is selective, so a
  wrong estimate is a common reason a good index goes unused.
