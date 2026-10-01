---
tags: [database, sql, schema-design, keys, indexing, postgresql, interview]
status: draft
author: claude
up: ["[[Database MOC]]"]
source: "https://www.postgresql.org/docs/15/sql-createindex.html"
created: 2026-10-01
score: 0.874
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 2
---
# Random UUID primary keys make a B-tree index larger than keys inserted in order

## Core idea
A B-tree index keeps its keys sorted, so where an insert lands depends on the key's value. Keys
that arrive in increasing order, such as a `bigint` identity or a time-ordered UUID, all go to the
rightmost leaf page, and PostgreSQL fills leaf pages to the index fillfactor when it extends the
index at the right. Keys that arrive in random order, such as version 4 UUIDs from
`gen_random_uuid()`, go to random leaf pages: full pages split all over the index and the split
pages stay partly empty, which the PostgreSQL documentation calls fragmentation of the on-disk
index structure. The same set of keys therefore needs more index pages when inserted in random
order, and each insert touches a page that is less likely to be cached. Version 7 UUIDs are
time-ordered, so they can be generated without coordination and still insert near the right edge
like a sequence; PostgreSQL 18 generates them with `uuidv7()`.

## Why choose / why not
- Choose a time-ordered key, a `bigint` identity or a UUID version 7, when: the table is
  write-heavy; inserts stay on the right edge of the index, which stays in memory.
- Choose UUID version 7 over `bigint` when: ids are generated outside the database, such as by
  several services or offline clients; it keeps the insert order without a central sequence.
- Accept random version 4 UUIDs when: an id must not reveal when it was created and the insert rate
  is modest; version 7 encodes its creation time in its leading bits, and the larger index is the
  price of hiding it.

## Interview angle
- Probed as "`bigint` or UUID for the primary key?".
- Common wrong answer: "UUIDs are always better for distributed systems."
- Strong answer: name the version and the insert order: random version 4 splits pages across the
  whole index, version 7 inserts in time order but leaks the creation time; back it with a
  measurement, such as one test on PostgreSQL 15.19 where the same 50,000 UUIDs gave a 275-page
  primary-key index in random insert order against 194 pages in key order.

## Related
- [[A B-tree index speeds lookups and range scans at the cost of slower writes and more storage]]:
  it states the storage cost of an index, and this note shows that the insert order of the keys
  changes that cost for the same set of keys.
