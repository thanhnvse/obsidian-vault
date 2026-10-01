---
tags: [database, concurrency, locking, jpa, interview]
status: draft
author: claude
source: "https://jakarta.ee/specifications/persistence/3.2/apidocs/jakarta.persistence/jakarta/persistence/version"
created: 2026-09-30
score: 0.862
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A version column detects a lost update at write time instead of blocking the other writer

## Core idea
Optimistic locking adds a version column that is read with the row and checked when the row is
written back: `UPDATE product SET price = ?, version = version + 1 WHERE id = ? AND version = ?`.
If another admin committed a change in between, the version no longer matches, the update
matches zero rows, and the writer learns that its copy was stale instead of silently
overwriting the other change. No lock is held between the read and the write, so the other
admin is never blocked; the cost is that the loser must reload and retry, or report a conflict.
In Jakarta Persistence 3.2, a `@Version` attribute enables this check, and a failed check makes
the provider throw `OptimisticLockException` and mark the active transaction for rollback.

## Why choose / why not
- Choose a version column when: conflicts are rare and the gap between read and write is long,
  such as an admin editing a product form for minutes; a lock held that long would block every
  other editor.
- Choose `SELECT ... FOR UPDATE` instead when: many writers contend for the same row at once,
  such as a hot stock counter; with a version column most of them would fail and retry.
- Choose a conditional `UPDATE` without a version when: the rule fits in the `WHERE` clause,
  such as `stock >= ?`; it needs no earlier read and no retry loop.

## Interview angle
- Probed as "two admins edit the same product; how do you stop one silently overwriting the
  other?"
- Common wrong answer: "optimistic locking prevents conflicts"; it only detects them, and the
  code must handle the failure.
- Strong answer: show the versioned `UPDATE`, say what the loser sees (an HTTP 409 or a reload
  and retry), and say when you would switch to a row lock.

## Related
- [[A synchronized block cannot prevent a lost update between two application instances]]: the
  version column is one of the database-side fixes that note calls for, and it holds across any
  number of instances.
