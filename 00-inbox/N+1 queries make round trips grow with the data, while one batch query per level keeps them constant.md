---
tags: [database, sql, performance, jdbc, interview]
status: draft
author: claude
up: ["[[Indexes and query performance MOC]]"]
source: "https://docs.hibernate.org/orm/5.2/userguide/html_single/chapters/fetching/Fetching.html"
created: 2026-10-01
score: 0.893
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# N+1 queries make round trips grow with the data, while one batch query per level keeps them constant

## Core idea
The N+1 pattern loads N parent rows with one query, then runs one more query per parent to load
its children: 1 + N statements. Each statement is a round trip to the database, so the page gets
slower as the data grows even when every single statement is fast. Loading the children of all
parents with one query by their ids, such as `WHERE order_id = ANY(?)`, needs two statements
whatever N is; a join needs one, but joining two child collections returns their Cartesian
product for each parent. With plain JDBC on PostgreSQL 15.19, loading 20 orders with their lines
took 21 statements one by one, 2 with the batch query and 1 with the join, and all three returned
the same orders and lines. Hibernate's documentation describes the same effect for lazy collections: the more parents the
first query fetches, the more secondary `SELECT` statements initialise their collections.

## Why choose / why not
- Load children with one batch query by ids when: a list page shows several parents with their
  children; it stays at one statement per level of the graph and returns no duplicated rows.
- Use a join when: there is a single child collection and the page is small; one round trip is
  cheapest, and the repeated parent columns are few.
- Keep one query per parent when: only one or two parents are ever loaded at a time, such as a
  detail page; the extra round trip is cheaper than a more complex query.

## Interview angle
- Probed as "every query is fast but the page is slow"; the answer is to count the statements per
  request, not to look for the slowest query.
- Common wrong answer: "join everything in one query"; two collections multiply the rows, and a
  `LIMIT` then applies to joined rows instead of parents.
- Strong answer: name the 1 + N count, the batch-by-ids fix, the join's row multiplication, and a
  test that asserts the statement count.

## Related
- [[EXPLAIN ANALYZE shows actual rows next to the planner's estimates for every plan node]]: that
  note reads one statement's plan; N+1 is the case where every plan is fine and the number of
  statements is the problem.
