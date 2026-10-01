---
tags: [moc, database, sql, interview]
type: moc
status: draft
author: claude
up: ["[[Java backend interview MOC]]"]
created: 2026-09-30
---
# Database MOC

The question behind this map: *what does the database guarantee, and what does it cost?*

## Modelling
- [[Cardinality decides where a relationship's foreign key goes]]: ER design in one rule
- [[Third normal form removes update anomalies by making every non-key column depend only on the key]]: why normalise
- [[Denormalise a read path only with a mechanism that keeps the copies consistent]]: when and how to break the rule

## Indexes and performance
- [[Indexes and query performance MOC]]: B-tree trade-offs, column order, index-only scans and reading a plan

## Isolation
- [[READ COMMITTED gives each statement its own snapshot, so two reads in one transaction can disagree]]: the default most services run on
- [[Write skew survives snapshot isolation because the two transactions write different rows]]: the anomaly that snapshots do not stop

## Related maps
- [[Concurrency MOC]]: lost updates and locking, the application side of isolation
