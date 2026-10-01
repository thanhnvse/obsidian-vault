---
tags: [database, sql, pagination, postgresql, interview]
status: draft
author: claude
up: ["[[Indexes and query performance MOC]]"]
source: "https://www.postgresql.org/docs/15/queries-limit.html"
created: 2026-10-01
score: 0.875
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# OFFSET pagination skips or repeats rows when rows before the page change between requests

## Core idea
Each page of `ORDER BY id LIMIT 10 OFFSET 10` is a separate query, and under READ COMMITTED each
query sees the data committed before it began. `OFFSET` counts rows to skip; it does not remember
which row the previous page ended on. If a row before the page is deleted between page 1 and page
2, every later row moves up by one place, so `OFFSET 10` starts one row later and the row that
slid onto page 1 is never shown. If a row is inserted before the page, such as a new post at the
top of a newest-first feed, every later row moves down by one, so the last row of page 1 is shown
again at the top of page 2. Keyset pagination continues from the sort key of the last row shown,
`WHERE id > :last_id`, so its position is a key rather than a count, and the same changes neither
skip nor repeat a row. On PostgreSQL 15.19, with 30 posts and pages of 10, deleting post 3 after
page 1 made the OFFSET page 2 run from post 12 to 21, so post 11 was never shown, while the keyset
page 2 ran from 11 to 20.

## Why choose / why not
- Accept OFFSET when: the data barely changes while users page, such as an admin list of
  reference data, or users must jump to page N; a rare skipped row costs nothing there.
- Use keyset pagination when: rows are inserted or deleted while clients page through them, such
  as a feed, an inbox or an export that must emit every row exactly once.
- Use one snapshot instead when: a long export must see the data as of one moment, such as a
  server-side cursor inside one REPEATABLE READ transaction; it holds a connection and a snapshot
  for the whole export.

## Interview angle
- Probed as "OFFSET or keyset?", after the candidate has given the performance answer.
- Common wrong answer: "OFFSET is only a performance problem; with an index it is fine."
- Strong answer: OFFSET is also wrong under concurrent changes, because the offset is a count of
  rows that have moved; keyset uses a key, so a delete or insert before the page shifts nothing.

## Related
- [[Keyset pagination stays fast on deep pages because it does not read the skipped rows]]: that
  note argues keyset pagination for speed; this one argues it for correctness, which holds even on
  the first pages.
- [[READ COMMITTED gives each statement its own snapshot, so two reads in one transaction can disagree]]:
  each page request is a new statement with a new snapshot, which is why the rows before the page
  can change between pages.
