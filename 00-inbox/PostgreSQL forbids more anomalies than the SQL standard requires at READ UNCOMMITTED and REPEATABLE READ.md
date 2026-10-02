---
tags: [database, isolation, transactions, postgresql, interview]
status: draft
author: claude
up: ["[[Database MOC]]"]
source: "https://www.postgresql.org/docs/15/transaction-iso.html"
created: 2026-10-01
score: 0.83
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# PostgreSQL forbids more anomalies than the SQL standard requires at READ UNCOMMITTED and REPEATABLE READ

## Core idea
The SQL standard's isolation table is a floor: it says which anomalies each level must not allow,
and a database may forbid more. PostgreSQL 15 forbids more at two levels, and both follow from its
snapshot-based MVCC. A PostgreSQL snapshot never contains changes that other transactions have not
committed, so READ UNCOMMITTED behaves like READ COMMITTED and never returns uncommitted rows,
although the standard allows dirty reads there; the manual calls this the only sensible way to map
the standard levels onto its MVCC architecture. REPEATABLE READ answers every query from one
transaction-wide snapshot, so PostgreSQL's REPEATABLE READ does not allow the phantom reads the
standard permits at that level; the manual notes that higher guarantees are acceptable under the
standard. So "READ UNCOMMITTED lets you read uncommitted data" and "REPEATABLE READ allows phantoms"
describe the standard's floor, not PostgreSQL.

## Why choose / why not
- Rely on the stronger guarantee when: the code only ever runs on PostgreSQL, such as a report that
  repeats a range query at REPEATABLE READ and needs the same rows both times.
- Don't rely on it when: the same code or its tests run on another engine; the standard's table is
  then the only portable guarantee, and MySQL's READ UNCOMMITTED really shows uncommitted rows.
- Don't request READ UNCOMMITTED in PostgreSQL to gain speed: it runs as READ COMMITTED, so it
  changes nothing.

## Interview angle
- Probed as "what does READ UNCOMMITTED allow?" or "does REPEATABLE READ prevent phantoms?"
- Common wrong answer: reciting the standard's table as if it described the database in use.
- Strong answer: the table is a minimum; PostgreSQL maps READ UNCOMMITTED to READ COMMITTED and its
  REPEATABLE READ is snapshot isolation with no phantoms; name the vendor before answering.

## Related
- [[READ COMMITTED gives each statement its own snapshot, so two reads in one transaction can disagree]]:
  a READ UNCOMMITTED transaction in PostgreSQL gets exactly those per-statement snapshots.
- [[Write skew survives snapshot isolation because the two transactions write different rows]]:
  being stricter than the standard at REPEATABLE READ still does not make it serializable.
- [[MySQL InnoDB REPEATABLE READ applies UPDATE and DELETE to the latest committed rows, not to the snapshot]]:
  another case where one level name means different behaviour per vendor.
