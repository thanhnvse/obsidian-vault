---
tags: [moc, database, sql, indexing, interview]
type: moc
status: draft
author: claude
up: ["[[Database MOC]]"]
created: 2026-09-30
---
# Indexes and query performance MOC

The question behind this map: *how does the database find rows, and how do I see what it did?*

## Indexes and performance
- [[A B-tree index speeds lookups and range scans at the cost of slower writes and more storage]]: the base trade-off
- [[A composite B-tree index is most efficient when the query constrains its leading columns]]: column order
- [[A covering index lets PostgreSQL answer a query with an index-only scan]]: avoiding the table
- [[EXPLAIN ANALYZE shows actual rows next to the planner's estimates for every plan node]]: how to read a slow query
- [[Keyset pagination stays fast on deep pages because it does not read the skipped rows]]: why OFFSET degrades
