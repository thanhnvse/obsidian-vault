---
tags: [database, sql, joins, postgresql, interview]
status: draft
author: claude
up: ["[[Database MOC]]"]
source: "https://www.postgresql.org/docs/15/queries-table-expressions.html"
created: 2026-10-07
review: unjudged
---
# A WHERE condition on the optional side of a LEFT JOIN drops the unmatched rows, so the join acts as an INNER JOIN unless the condition tests for NULL

## Core idea
A `LEFT JOIN` is an inner join plus every left row that found no partner, padded with `NULL` on
the right. `WHERE` runs after the join and keeps a row only when its condition is true. Take
`employee e LEFT JOIN department d ON d.id = e.dept_id`, where the department is the optional side.
For the padded row, the one employee with no department, `d.name = 'Sales'` is `NULL = 'Sales'`,
which is unknown, so the row is dropped and the result is what an `INNER JOIN` with the same
`WHERE` returns; the PostgreSQL 15 plan then contains an inner join too. With nine employees that
left join returns 9 rows, `WHERE d.name = 'Sales'` leaves 3 (the padded row and the five employees
of other departments are gone), and the same test moved into `ON` keeps all 9. Only a condition
that is true for the padded row keeps it, and the anti-join relies on that: with the tables the
other way round, in `department d LEFT JOIN employee e ON e.dept_id = d.id`, `WHERE e.id IS NULL`
keeps only the padded rows and finds Legal, the department with no employees.

## Why choose / why not
- Put the condition in `ON` when it decides which right-hand rows may match and every left row
  must stay: the unmatched ones show `NULL` (`ON d.id = e.dept_id AND d.name = 'Sales'` returns 9).
- Keep it in `WHERE` when you really want only the matched rows: the result is that of an inner
  join.
- Add `OR d.id IS NULL` when the condition must stay in `WHERE` but the unmatched rows must
  survive: `WHERE d.name <> 'Legal'` returns 8 of 9 employees, because `NULL <> 'Legal'` is
  unknown too, and with `OR d.id IS NULL` it returns all 9.
- To find the rows with no partner when you already have the join for other columns, test for
  `NULL` in `WHERE` a column that is `NOT NULL` in the right table, such as its key; the wrong
  column is easy to pick. For the other spelling of that question, see [[NOT IN returns no rows when its subquery yields a NULL, so NOT EXISTS is the safe test for rows with no match]].

## Interview angle
- Probed as "My `LEFT JOIN` lost rows. Why?"; the cause is almost always a `WHERE` on the
  right-hand table.
- Common wrong answer: "A `LEFT JOIN` always returns every row of the left table."
- Strong answer: the padded row has `NULL` there and `WHERE` keeps only true, so the join acts as
  an inner join; put the condition in `ON`, or test the key for `NULL` as well, and add that the
  planner makes the same deduction.

## Related
- [[Database MOC]]: the query runs without an error and returns plausible rows, so this is
  database behaviour a result can silently depend on, which is what the map's question about
  guarantees and costs covers.
- [[NOT IN returns no rows when its subquery yields a NULL, so NOT EXISTS is the safe test for rows with no match]]:
  `NOT EXISTS` and `LEFT JOIN ... WHERE e.id IS NULL` return the same rows for "has no partner";
  that note explains why `NOT IN` is the spelling to avoid.

Written up in win-interview: backend/docs/sql-by-hand.md, sections 2.2 and 2.5
