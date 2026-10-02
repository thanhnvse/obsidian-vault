---
tags: [database, sql, indexing, collation, postgresql, interview]
status: draft
author: claude
up: ["[[Indexes and query performance MOC]]"]
source: "https://www.postgresql.org/docs/15/indexes-opclass.html"
created: 2026-10-01
score: 0.887
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Under a non-C collation a prefix LIKE cannot use a default PostgreSQL B-tree index

## Core idea
PostgreSQL can serve `col LIKE 'abc%'` from a B-tree index when the pattern is a constant anchored
at the start of the string. A default B-tree index on a text column is sorted by the column's
collation, and when the database does not use the C locale, such as `en_US.utf8`, the default
index cannot serve the pattern match, so the query reads the table. An index created with the
`text_pattern_ops` operator class compares values strictly character by character instead of by
the locale's rules, so it serves the prefix `LIKE`. It cannot serve ordinary `<`, `<=`, `>` or
`>=` comparisons, so a column that is also range-compared needs a second index with the default
operator class. A leading wildcard, `LIKE '%abc'`, is no range in any B-tree. On PostgreSQL 15.19
with database collation `en_US.utf8`, `email LIKE 'User4242@%'` was a sequential scan with a
default index on `email`, and an index scan with an index on `email text_pattern_ops`.

## Why choose / why not
- Add a `text_pattern_ops` index when: a hot query searches by prefix, such as autocomplete on a
  product code, and the database collation is not C.
- Declare the column `COLLATE "C"` instead when: it holds identifiers such as SKUs or codes that
  nobody sorts linguistically; one default index then serves equality, prefix `LIKE` and
  `ORDER BY`.
- Use a trigram GIN index from `pg_trgm` instead when: users search in the middle of the string;
  no B-tree operator class serves a leading wildcard.

## Interview angle
- Probed as "`LIKE 'abc%'` uses the index on my laptop but not in production"; the two databases
  were created with different collations.
- Common wrong answer: "a B-tree index always serves a prefix `LIKE`."
- Strong answer: it depends on the collation; under a linguistic collation use `text_pattern_ops`
  or a C-collated column, and keep the default index too if the column is also range-compared.

## Related
- [[Wrapping an indexed column in a function hides it from a plain PostgreSQL index]]: both are
  cases of the same rule, that an index only serves conditions that match what it stores and the
  order it stores it in.
- [[A B-tree index speeds lookups and range scans at the cost of slower writes and more storage]]:
  a prefix `LIKE` is served as a range scan of that B-tree, and the collation decides whether the
  prefix is one contiguous range.
