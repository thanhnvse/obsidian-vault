---
tags: [database, sql, indexing, postgresql, interview]
status: draft
author: claude
up: ["[[Indexes and query performance MOC]]"]
source: "https://www.postgresql.org/docs/16/storage-hot.html"
created: 2026-10-01
score: 0.887
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# In PostgreSQL an UPDATE of one indexed column adds a new entry to every index on the table

## Core idea
PostgreSQL carries out an `UPDATE` by writing a new version of the row, and every index entry
points at a row version. When the update changes no column that any index covers and the new
version fits on the same heap page, it is a heap-only tuple (HOT) update: no index gets a new
entry, and a lookup follows a chain from the old version to the new one. When the update changes
any indexed column, the new version needs a new entry in every index on the table, including the
indexes whose columns did not change. On PostgreSQL 15.19, a table with three indexes and ten rows
went to eleven entries in each index after one update of the indexed `email` column, and stayed
at ten after one update of the unindexed `balance` column.

## Why choose / why not
- Keep a frequently updated column out of indexes when: the table already has several indexes;
  indexing a column such as `status` or `updated_at` turns its HOT updates into a write to every
  index.
- Lower the table's `fillfactor` when: updates are frequent and touch only unindexed columns;
  free space on the old row's page is the second condition for a HOT update.
- Don't tune for HOT when: the table is append-only; every insert writes every index anyway.

## Interview angle
- Probed as "what does one more index cost?"; a senior answer adds that indexing a frequently
  updated column also disables HOT for those updates, so every other index pays too.
- Common wrong answer: "an update only touches the index of the column it changes."
- Strong answer: new row version, new entry in each index unless the update is HOT; check
  `n_tup_hot_upd` against `n_tup_upd` in `pg_stat_user_tables`.

## Related
- [[A B-tree index speeds lookups and range scans at the cost of slower writes and more storage]]:
  that note prices one index on insert; this one shows an update of one indexed column paying that
  price in every index on the table.
