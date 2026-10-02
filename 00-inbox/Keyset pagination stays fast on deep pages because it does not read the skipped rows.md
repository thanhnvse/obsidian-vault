---
tags: [database, sql, performance, pagination, postgresql, interview]
status: draft
author: claude
source: "https://www.postgresql.org/docs/18/queries-limit.html"
created: 2026-09-30
score: 0.861
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Keyset pagination stays fast on deep pages because it does not read the skipped rows

## Core idea
`LIMIT 20 OFFSET 100000` still makes PostgreSQL compute the 100 000 skipped rows before it
returns 20, so every page is slower than the one before it. Keyset pagination, also called the
seek method, remembers the sort key of the last row returned and asks only for rows after it:
`WHERE (created_at, id) < (?, ?) ORDER BY created_at DESC, id DESC LIMIT 20`. With an index on
`orders(created_at, id)`, the scan starts at that position and reads about 20 entries however
deep the page is. Appending the unique `id` to the sort key makes the order deterministic, so
rows that share a `created_at` value are neither skipped nor repeated between pages.

## Why choose / why not
- Choose keyset pagination when: clients move forward through a large or growing list, such as
  an order feed or a batch export; each page costs the same, and rows inserted at the top do
  not shift later pages the way they do with OFFSET.
- Keep OFFSET when: the list is short or users must jump straight to page 37; keyset pagination
  can only move to the next or previous page.
- Don't switch to keyset without an index matching the `ORDER BY`: without it, each page still
  sorts the whole result set.

## Interview angle
- Probed as "page 5 000 of our order API is slow; why?"; the answer is that OFFSET reads and
  discards every row before the page.
- Common wrong answer: "add an index on `created_at`" while keeping the large OFFSET; the index
  removes the sort, but the skipped rows are still read.
- Strong answer: return a cursor made of the last row's `(created_at, id)`, filter on it with a
  row comparison, back it with a matching composite index, and name the trade-off: no jumping
  to page N.

## Related
- [[A composite B-tree index is most efficient when the query constrains its leading columns]]:
  keyset pagination only stays cheap when the index's leading columns match the `ORDER BY`,
  which is that leading-column rule applied to sorting.
