---
tags: [database, migration, rollback, ops, interview]
status: draft
author: claude
up: ["[[Ops and cloud MOC]]"]
source: "https://documentation.red-gate.com/fd/undo-migrations-273973334.html"
created: 2026-10-01
score: 0.743
review: "borderline"
score_reasons: ["atomic: 0.45 (borderline)", "numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 2
---
# A Flyway undo migration can reverse a schema change but not lost data, so production databases roll forward

## Core idea
Redgate's Flyway documentation (undo migrations page, last updated 21 May 2026, Teams edition)
describes undo migrations: `U` scripts that undo the effects of the versioned migration with the
same version. An undo script reverses structure more reliably than content: undo migrations work
for undoing schema changes but not so well for undoing data changes, because after a destructive
change such as a drop, delete or truncate the undo script would have to restore both the table and
its data, which is challenging unless the data is static. For that reason Flyway's rollback-strategy
page states that when reverting schema changes on a live production database, it is not only
simpler to roll forward with a new migration, it also keeps the deployment audit trail.

## Why choose / why not
- Roll forward with a new migration when: the production schema has already changed and the new
  version has written data; it is simpler and keeps the audit trail.
- Use an undo script when: a schema-only change fully succeeded and wrote no data you need, for
  example in a development or test database; an undo script assumes the whole migration
  succeeded, so it does not help after a partial failure on a database without DDL transactions.
- Restore from a backup only when: a destructive migration dropped data; everything written after
  the backup is lost unless it is replayed.

## Interview angle
- Probed as "how do you roll back a database migration?"
- Common wrong answer: "we run the undo script."
- Strong answer: design migrations so the previous release still runs, roll the code back and fix
  forward; name the two limits of undo scripts; keep a tested backup as the last resort.

## Related
- [[Expand and contract schema changes keep the previous version runnable after a rollback]]: that
  pattern is why the schema rarely needs to go back; this note covers what to do when it would.
- [[kubectl rollout undo rolls back only the Deployment's Pod template]]: a code rollback leaves the
  schema in place, which is the moment the choice between undo and roll forward comes up.
