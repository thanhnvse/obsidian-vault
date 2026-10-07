---
tags: [database, sql, transactions, postgresql, interview]
status: draft
author: claude
up: ["[[Database MOC]]"]
source: "https://www.postgresql.org/docs/15/sql-truncate.html"
created: 2026-10-07
review: unjudged
---
# TRUNCATE rolls back in PostgreSQL, but MySQL and Oracle commit it implicitly, so it cannot be undone there

## Core idea
In PostgreSQL `TRUNCATE` is transaction-safe: the manual says the truncation is safely rolled back
if the surrounding transaction does not commit. In the lab, `TRUNCATE contact` left 0 rows inside the
transaction and `ROLLBACK` brought all 8 back; `DELETE` and `DROP TABLE` roll back too, but a
committed `TRUNCATE` is final. The other databases differ, and these two claims come from their
manuals and were not run: MySQL 8.0 says truncate operations cause an implicit commit and cannot be
rolled back ([MySQL TRUNCATE TABLE](https://dev.mysql.com/doc/refman/8.0/en/truncate-table.html)),
and Oracle 19c says a `TRUNCATE TABLE` cannot be rolled back
([Oracle TRUNCATE TABLE](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/TRUNCATE-TABLE.html))
and that it implicitly commits the current transaction before and after every DDL statement, which
includes `TRUNCATE`
([Oracle statement types](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/Types-of-SQL-Statements.html)).

## Why choose / why not
- Use `TRUNCATE` when: you empty a table you own, such as staging data or test data between tests;
  it does not scan the table and returns the space at once.
- Use `DELETE` when: readers must keep working, or you need `WHERE`, row triggers or per-row foreign
  key checks; `TRUNCATE` takes `ACCESS EXCLUSIVE`, which blocks even a `SELECT` and waits for open
  readers.
- Don't expect it to be MVCC-safe: a `REPEATABLE READ` transaction whose snapshot was taken before a
  committed `TRUNCATE` sees an empty table, while a committed `DELETE` leaves its snapshot intact.
- Don't add `CASCADE` casually: it empties the referencing tables too, and without it PostgreSQL
  refuses a table that another table references unless both are in the same command.
- Don't plan on the rollback outside PostgreSQL: on MySQL and Oracle the statement commits, so
  `ROLLBACK` cannot bring the rows back.

## Interview angle
- Probed as "can you roll back a `TRUNCATE`?" or "`DELETE`, `TRUNCATE` and `DROP`: what is the
  difference?".
- Common wrong answer: "`TRUNCATE` cannot be rolled back", which is true for MySQL and Oracle but
  false for PostgreSQL, or "it is just a faster `DELETE`".
- Strong answer: name the database first; in PostgreSQL `BEGIN; TRUNCATE ...; ROLLBACK;` restores the
  rows, in MySQL it commits first, and Oracle documents that it cannot be rolled back; then add
  `ACCESS EXCLUSIVE` and not MVCC-safe.

## Related
- [[Database MOC]]: the map for database interview questions; this is the question where the answer
  changes with the database in front of you.
- [[A Flyway undo migration can reverse a schema change but not lost data, so production databases roll forward]]:
  that note says an undo script cannot restore data after a destructive change; here the commit is
  what makes `TRUNCATE` final, and on MySQL and Oracle the statement commits by itself.

Written up in win-interview: backend/docs/sql-by-hand.md, sections 2.8 (DELETE, TRUNCATE and DROP) and 3.4 (Dialect differences)
