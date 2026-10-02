---
tags: [database, migration, rollback, ops, interview]
status: draft
author: claude
source: "https://martinfowler.com/bliki/ParallelChange.html"
created: 2026-09-30
score: 0.834
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Expand and contract schema changes keep the previous version runnable after a rollback

## Core idea
Parallel change, also called expand and contract, makes a backward-incompatible change safely in
three phases: expand, migrate and contract. Applied to a database schema, expand adds the new
structure next to the old one, migrate moves every client to the new structure, and contract
removes the old structure only once nothing uses it. Between expand and contract the schema
supports both the old and the new version of the application. Rolling back the application
restores the old code but not the old schema, so this window is what lets the previous version
still run after a rollback.

## Why choose / why not
- Choose when: a change would break the version that is still running, such as renaming or
  splitting a column, changing its type or making it NOT NULL, and you deploy with rolling
  updates or need a rollback path.
- Don't choose when: the change is purely additive, like a new table or a nullable column; that
  is already backward compatible, so one migration is enough.
- Budget for: at least two or three releases, a backfill or dual writes during the migrate step,
  and the discipline to actually run the contract step later.

## Interview angle
- Probed as "how do you rename a column with zero downtime?"; the answer is a sequence of
  releases, not one migration.
- Common wrong answer: "run the Flyway migration at startup and deploy." During the rolling
  update the old Pods still query the old column name and fail.
- Strong answer: walk the releases: add the new column and write both, backfill, switch reads,
  then drop the old column; each step keeps the schema compatible with the release before it,
  so rolling back one release is safe.

## Related
- [[kubectl rollout undo rolls back only the Deployment's Pod template]]: that rollback restores
  old code but leaves the schema as it is, which is why the schema must stay compatible.
- [[Blue-green switches all traffic at once while a canary shifts a subset of users first]]:
  both strategies run the old and new version against one database, so they depend on this
  pattern too.
