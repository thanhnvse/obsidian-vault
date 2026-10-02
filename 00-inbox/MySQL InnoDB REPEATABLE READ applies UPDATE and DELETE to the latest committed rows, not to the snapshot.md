---
tags: [database, isolation, transactions, mysql, interview]
status: draft
author: claude
up: ["[[Database MOC]]"]
source: "https://dev.mysql.com/doc/refman/8.4/en/innodb-consistent-read.html"
created: 2026-09-30
score: 0.89
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# MySQL InnoDB REPEATABLE READ applies UPDATE and DELETE to the latest committed rows, not to the snapshot

## Core idea
At InnoDB's default REPEATABLE READ in MySQL 8.4, plain `SELECT`s read the snapshot established by
the transaction's first read, but "the snapshot of the database state applies to SELECT statements
within a transaction, not necessarily to DML statements". An `UPDATE` or `DELETE` works on the
latest committed rows: the manual's own example counts 0 matching rows with a `SELECT`, and the
next `UPDATE` in the same transaction changes 10 rows another transaction had just committed. So a
value computed from a snapshot read and written back can overwrite a newer committed value.
PostgreSQL's REPEATABLE READ would reject that write with SQLSTATE `40001` instead. The manual's
way to read the freshest data is READ COMMITTED or a locking read (`FOR UPDATE` or `FOR SHARE`).

## Why choose / why not
- Use a locking read (`SELECT ... FOR UPDATE`) when: an InnoDB transaction decides a write from a
  value it read, such as a stock check before a sale.
- Write relative updates (`SET stock = stock - 1`) when: the change can be computed in SQL; the
  statement then works on the latest row.
- Don't assume PostgreSQL's REPEATABLE READ behaviour when: porting code or tests between the two
  databases; the same level name protects different things.

## Interview angle
- Probed as "is REPEATABLE READ in MySQL the same as in PostgreSQL?"
- Common wrong answer: "same name, same guarantees."
- Strong answer: InnoDB's snapshot covers plain reads only, DML and locking reads see the latest
  committed rows, and PostgreSQL instead aborts a write on a row changed after its snapshot.

## Related
- [[READ COMMITTED gives each statement its own snapshot, so two reads in one transaction can disagree]]:
  that note names REPEATABLE READ as MySQL's default; this one shows the name hides a different
  write behaviour.
