---
tags: [database, sql, denormalization, postgresql, interview]
status: draft
author: claude
up: ["[[Database MOC]]"]
source: "https://www.postgresql.org/docs/15/sql-refreshmaterializedview.html"
created: 2026-10-01
score: 0.877
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A PostgreSQL materialized view shows the data of its last refresh, not the current data

## Core idea
A materialized view stores the result of its query when it is created or refreshed, and does not
follow later changes to the tables it reads. Its data is therefore not always current: a row
inserted into a base table is invisible in the view until `REFRESH MATERIALIZED VIEW` runs the
whole query again and replaces the contents. A plain refresh can block other connections that try
to read the view while it runs. `REFRESH MATERIALIZED VIEW CONCURRENTLY` lets reads continue, but
it is only allowed when the view has at least one `UNIQUE` index, and it is slower when many rows
change. On PostgreSQL 15.19 an order inserted after a revenue view was built was missing from the
view until it was refreshed.

## Why choose / why not
- Choose a materialized view when: an expensive aggregate is read often and readers accept data as
  of the last refresh, such as a revenue dashboard refreshed every fifteen minutes.
- Refresh `CONCURRENTLY` when: readers must not wait during the refresh; create a `UNIQUE` index
  on the view first, and accept a slower refresh.
- Don't use one when: a reader must see its own write, such as an order confirmation page; keep a
  copy maintained in the same transaction, or query the base tables.
- Prefer an incrementally updated rollup table when: the view is large and only a small part of it
  changes between refreshes; every refresh recomputes the whole query.

## Interview angle
- Probed as "how would you make this dashboard query fast?", then "how fresh is the number?".
- Common wrong answer: "a materialized view is a cached table that stays up to date."
- Strong answer: stale until refreshed, every refresh recomputes the whole query, a plain refresh
  can block readers, `CONCURRENTLY` needs a unique index; then state the staleness the dashboard
  accepts.

## Related
- [[Denormalise a read path only with a mechanism that keeps the copies consistent]]: a
  materialized view is one of the mechanisms it names, the one whose copy is stale between
  refreshes.
