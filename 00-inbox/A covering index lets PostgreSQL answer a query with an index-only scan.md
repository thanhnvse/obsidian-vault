---
tags: [database, sql, indexing, postgresql, interview]
status: draft
author: claude
source: "https://www.postgresql.org/docs/18/indexes-index-only-scans.html"
created: 2026-09-30
score: 0.857
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A covering index lets PostgreSQL answer a query with an index-only scan

## Core idea
An ordinary index scan finds the matching entries in the index and then fetches each row from
the table heap, often with random reads. When the index holds every column the query
references, PostgreSQL 18 can use an index-only scan and return the values from the index
alone. It must still know whether each row is visible to the query's snapshot, and it reads that
from the table's visibility map: a heap visit is skipped only for pages whose rows are all
visible, so the win depends on the table changing slowly. `CREATE INDEX ON orders (customer_id)
INCLUDE (total)` makes the index cover `SELECT total FROM orders WHERE customer_id = ?` while
keeping `total` out of the search key.

## Why choose / why not
- Choose a covering index when: a hot query reads one or two narrow columns by a selective key,
  such as the totals of one customer's orders; `INCLUDE (total)` removes the heap visit for
  each matching row.
- Don't cover when: the table is updated heavily; recently changed pages are not all-visible,
  so the scan visits the heap anyway and the index is only bigger.
- Don't include wide columns: non-key columns duplicate table data and bloat the index, which
  slows searches; put a column in the key instead when the query also filters or sorts by it.

## Interview angle
- Probed through a plan: an "Index Only Scan" with a high "Heap Fetches" count means the
  visibility map, not the index definition, is the problem.
- Common wrong answer: "an index-only scan never touches the table."
- Strong answer: name the columns the query needs, show that the index holds them, and say the
  saving depends on how many heap pages are all-visible.

## Related
- [[A composite B-tree index is most efficient when the query constrains its leading columns]]:
  a covering index adds returned columns to a searched one, so the leading-column rule still
  decides how narrow its scan is.
