---
tags: [database, sql, performance, statistics, postgresql, interview]
status: draft
author: claude
up: ["[[Indexes and query performance MOC]]"]
source: "https://www.postgresql.org/docs/15/planner-stats.html"
created: 2026-10-01
score: 0.92
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Correlated columns make the PostgreSQL planner underestimate rows until extended statistics record the dependency

## Core idea
For `WHERE city = 'city-7' AND zip = 'zip-75'` the PostgreSQL planner estimates the selectivity of
each condition from that column's own statistics and, assuming the conditions are independent,
multiplies them. Per-column statistics cannot capture a correlation between columns. When one
column determines the other, as a zip code determines its city, the second condition removes
almost nothing, so the multiplied estimate is far too low. On PostgreSQL 15.19, with 30,000
addresses, 300 zip codes and 10 zip codes per city, the planner estimated 3 rows where 100
matched. `CREATE STATISTICS address_zip_city (dependencies) ON city, zip FROM address`, followed by
`ANALYZE address`, gives the planner the measured functional dependency, and the estimate became
100. Dependency statistics apply only to equality conditions and `IN` lists that compare columns
with constants.

## Why choose / why not
- Create dependency statistics when: a plan's estimated rows are far below the actual rows on a
  multi-column equality filter whose columns move together, such as zip and city, or country and
  currency.
- Don't expect them to help when: the conditions are ranges, or compare columns with each other;
  dependency statistics are only applied to equality and `IN` against constants.
- Don't create them for every column pair: each object adds `ANALYZE` work; start from a plan whose
  estimate is wrong.

## Interview angle
- Probed as "the plan estimates 3 rows and gets 100; why, and does it matter?".
- Common wrong answer: "the statistics are stale, run `ANALYZE`"; here they are fresh, and the
  independence assumption is what is wrong.
- Strong answer: the planner multiplies per-column selectivities; correlated columns break that;
  extended statistics fix it; and under a join the low estimate can pick a nested loop that runs
  far more often than planned.

## Related
- [[EXPLAIN ANALYZE shows actual rows next to the planner's estimates for every plan node]]: it
  finds the node where estimate and actual diverge, and this note explains a cause of that gap
  that `ANALYZE` alone does not fix.
