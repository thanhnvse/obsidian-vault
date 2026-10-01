---
tags: [database, concurrency, locking, postgresql, interview]
status: draft
author: claude
up: ["[[Concurrency MOC]]"]
source: "https://www.postgresql.org/docs/15/transaction-iso.html"
created: 2026-10-01
score: 0.906
review: "ready"
score_reasons: ["possible conflict with draft [[A version column detects a lost update at write time instead of blocking the other writer]] (p=0.68)", "numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# An optimistic version check still takes a row lock, so an overlapping writer waits and then updates zero rows

## Core idea
Optimistic locking holds no lock between the read and the write, but its versioned write,
`UPDATE item SET stock = ?, version = version + 1 WHERE id = ? AND version = ?`, is still an
`UPDATE`: in PostgreSQL 15 it takes a row lock on the row it changes and keeps it until its
transaction ends. A second writer that sends the same old version while the first transaction is
still open therefore waits for that lock. At READ COMMITTED, once the first transaction commits,
PostgreSQL re-evaluates the second `UPDATE`'s `WHERE` clause against the newest row version; the
version no longer matches, so the second `UPDATE` changes 0 rows instead of overwriting. If the
first transaction rolls back instead, the second `UPDATE` proceeds on the row it found. So
"optimistic" means no lock across the user's think time, not no locks at all.

## Why choose / why not
- Choose a version check when: the gap between read and write is long, such as an edit form; the
  only lock lives from the versioned `UPDATE` to the commit.
- Commit right after the versioned `UPDATE` when: the code could still do slow work, such as an
  HTTP call; anything after the `UPDATE` holds the row lock, and competing writers wait as they
  would behind `SELECT ... FOR UPDATE`.
- Don't expect the loser to fail fast when: writes overlap; it learns that it lost only after the
  winner commits, so add `lock_timeout` if that wait must be bounded.

## Interview angle
- Probed as "does optimistic locking use any locks?"
- Common wrong answer: "no locks at all, so writers never wait."
- Strong answer: nothing is held during the think time; the `UPDATE` itself locks the row until
  commit; an overlapping writer waits, then READ COMMITTED re-checks `version = ?` on the new row
  and finds 0 rows; the loser reloads and retries in a new transaction.

## Related
- [[A version column detects a lost update at write time instead of blocking the other writer]]:
  that note covers the version check when the writes do not overlap; this one adds the overlapping
  case, where the loser waits for the winner's row lock before it sees 0 rows.
- [[SELECT FOR UPDATE makes competing writers queue on the row until the lock holder commits]]: the
  same row lock, taken at write time instead of read time, which is why both approaches can make a
  writer wait.
- [[A conditional UPDATE with the stock check in its WHERE clause cannot oversell under READ COMMITTED]]:
  the same re-evaluation of the `WHERE` clause turns a stale version into 0 rows here and a sold-out
  item into 0 rows there.
