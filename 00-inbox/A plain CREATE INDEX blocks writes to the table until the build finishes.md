---
tags: [database, sql, indexing, migration, postgresql, interview]
status: draft
author: claude
up: ["[[Indexes and query performance MOC]]"]
source: "https://www.postgresql.org/docs/15/sql-createindex.html"
created: 2026-10-01
score: 0.843
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A plain CREATE INDEX blocks writes to the table until the build finishes

## Core idea
A standard `CREATE INDEX` in PostgreSQL locks the table against writes, but not reads, and builds
the index in one scan of the table. Other transactions can still read the table, but an
`INSERT`, `UPDATE` or `DELETE` on it blocks until the build has finished, so on a large live table
writers queue for the whole build. `CREATE INDEX CONCURRENTLY` builds the index without taking a
lock that prevents concurrent inserts, updates or deletes. In exchange it scans the table twice,
waits for every existing transaction that could modify or use the index to end, and cannot run
inside a transaction block. If it fails, for example on a uniqueness violation, it leaves behind
an invalid index that queries ignore but every write still maintains, so it should be dropped and
built again.

## Why choose / why not
- Use `CONCURRENTLY` when: the table takes writes while the migration runs; writers keep going and
  only the build takes longer.
- Use a plain `CREATE INDEX` when: the table is new or empty, or writes are stopped for a
  maintenance window; it is faster and can run inside the migration's transaction.
- Mark the script as non-transactional when: the migration tool wraps each script in a
  transaction; inside a transaction block `CONCURRENTLY` fails before it starts, so check how
  your tool runs one script outside it.
- Check for invalid indexes after a failed build: `pg_index.indisvalid` is false for them; drop
  and retry, because the leftover still costs every write.

## Interview angle
- Probed as "how do you add an index to a large table in production?".
- Common wrong answer: "indexes are free to add; just put `CREATE INDEX` in the migration."
- Strong answer: `CONCURRENTLY`, outside a transaction block; it is slower and waits for running
  transactions, and a failure leaves an invalid index to drop.

## Related
- [[Expand and contract schema changes keep the previous version runnable after a rollback]]: both
  are zero-downtime schema changes; that note keeps the old code running, and this one keeps the
  writers running during the index build.
- [[A B-tree index speeds lookups and range scans at the cost of slower writes and more storage]]:
  it prices an index once it exists, and this note is the cost of creating one on a live table.
