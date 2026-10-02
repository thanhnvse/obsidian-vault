---
tags: [database, isolation, transactions, concurrency, postgresql, interview]
status: draft
author: claude
up: ["[[Database MOC]]"]
source: "https://www.postgresql.org/docs/15/transaction-iso.html"
created: 2026-10-01
score: 0.893
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# PostgreSQL REPEATABLE READ rejects a lost update with SQLSTATE 40001 instead of overwriting the newer row

## Core idea
In PostgreSQL 15, a REPEATABLE READ transaction reads from one snapshot, taken at its first
statement, and it may not modify or lock a row that another transaction changed and committed
after that snapshot. When its `UPDATE`, `DELETE` or `SELECT ... FOR UPDATE` reaches such a row, it
fails with `could not serialize access due to concurrent update`, SQLSTATE `40001`, instead of
writing. If the other transaction is still open, the statement first waits for it: if the other
transaction commits, the waiting statement fails with `40001`; if it rolls back, the waiting
statement proceeds on the row it found. So when two sales both read stock 10 and both try to
write 9, the second one fails instead of erasing the first sale: the first updater wins. READ
COMMITTED, in the same situation, waits and then applies the write to the newest row version.

## Why choose / why not
- Choose REPEATABLE READ when: a transaction computes a new value in application code from what
  it read, and the caller can retry the whole transaction on `40001`; the database then refuses
  the stale write instead of losing an update.
- Don't choose it when: the code has no retry path; under contention users see errors where READ
  COMMITTED would have made them wait. Stay at READ COMMITTED and use an atomic `UPDATE`, a
  version column or `SELECT ... FOR UPDATE` for that one race.
- Don't rely on it when: the rule spans several rows, such as an on-call minimum; each
  transaction writes a different row, so first updater wins never fires.

## Interview angle
- Probed as "two transactions at REPEATABLE READ both read stock 10 and both write 9; what does
  PostgreSQL do?"
- Common wrong answer: "the second write waits and then overwrites, as at READ COMMITTED", or
  "the same as MySQL".
- Strong answer: first updater wins; the second gets `40001`, at once if the first already
  committed, after a wait if it is still open, and it proceeds if the first rolls back; the fix is
  a retry of the whole transaction; then contrast READ COMMITTED and InnoDB.

## Related
- [[MySQL InnoDB REPEATABLE READ applies UPDATE and DELETE to the latest committed rows, not to the snapshot]]:
  the same level name in InnoDB applies the stale write with no error, so this note is the
  PostgreSQL side of that contrast.
- [[Write skew survives snapshot isolation because the two transactions write different rows]]:
  first updater wins only fires when both transactions write the same row, which is exactly why
  write skew gets past it.
- [[A serialization failure must be retried as a new transaction that re-runs its reads]]: the
  `40001` this level raises is only useful with that retry around the transaction.
