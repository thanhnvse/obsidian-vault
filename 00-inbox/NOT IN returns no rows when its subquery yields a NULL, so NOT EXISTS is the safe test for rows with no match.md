---
tags: [database, sql, null, postgresql, interview]
status: draft
author: claude
up: ["[[Database MOC]]"]
source: "https://www.postgresql.org/docs/15/functions-subquery.html"
created: 2026-10-07
review: unjudged
---
# NOT IN returns no rows when its subquery yields a NULL, so NOT EXISTS is the safe test for rows with no match

## Core idea
When no right-hand value equals the left-hand one and at least one right-hand row is `NULL`,
`x NOT IN (subquery)` is null, not true, and `WHERE` keeps only true. For Legal (id 4),
`d.id NOT IN (1, 2, 3, NULL)` means `d.id <> 1 AND d.id <> 2 AND d.id <> 3 AND d.id <> NULL`:
three true and one unknown, so the row is dropped. `WHERE d.id NOT IN (SELECT e.dept_id FROM
employee e)` returns 0 rows although Legal has no employees, because one `dept_id` is `NULL`;
`WHERE NOT EXISTS (SELECT 1 FROM employee e WHERE e.dept_id = d.id)` returns Legal. A `NULL` on the left-hand side has the same effect: "employees
whose department does not exist" written with `NOT IN` returns nothing, while `NOT EXISTS` returns
the employee whose `dept_id` is `NULL`.

## Why choose / why not
- Choose `NOT EXISTS` for "has no match": it needs no care about `NULL`, and it is the default
  for the negative form.
- Use `NOT IN` only against a list that cannot contain `NULL`, or filter it inside the subquery
  with `WHERE e.dept_id IS NOT NULL`, which repairs the query.
- The danger is that `NOT IN` works until the day it does not: once the one employee without a
  department is assigned to Support the query returns Legal, and the next row with a `NULL`
  `dept_id` makes it return nothing, while `NOT EXISTS` still returns Legal.
- `LEFT JOIN ... WHERE e.id IS NULL` returns the same rows as `NOT EXISTS`, but the plans may
  differ, so measure; see [[A WHERE condition on the optional side of a LEFT JOIN drops the unmatched rows, so the join acts as an INNER JOIN unless the condition tests for NULL]].

## Interview angle
- Probed as "`NOT IN` returns no rows. What is wrong?"; the subquery returns a `NULL`, or the
  left-hand value is `NULL`.
- Common wrong answer: "`NOT IN` and `NOT EXISTS` are the same."
- Strong answer: `x NOT IN (..., NULL)` is unknown, never true, so use `NOT EXISTS` or filter
  `IS NOT NULL`; plain `IN` has no such problem because a match is still a match; the query
  passes review and tests while the data has no `NULL`, so put a `NULL` row in the test data.

## Related
- [[Database MOC]]: three-valued logic is a correctness behaviour of the database that a query
  can depend on silently, which is what the map's question about what the database guarantees
  covers.
- [[A WHERE condition on the optional side of a LEFT JOIN drops the unmatched rows, so the join acts as an INNER JOIN unless the condition tests for NULL]]:
  the same unknown-is-not-true rule makes a `WHERE` on the optional side drop the padded rows.
- [[A PostgreSQL UNIQUE constraint accepts several NULLs unless it is declared NULLS NOT DISTINCT]]:
  another place where two `NULL`s are not treated as equal: a unique constraint accepts several
  of them by default.

Written up in win-interview: backend/docs/sql-by-hand.md, section 2.5
