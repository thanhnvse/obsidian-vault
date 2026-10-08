---
tags: [concurrency, distributed-systems, redis, locking, interview]
status: draft
author: claude
up: ["[[Concurrency MOC]]"]
source: "https://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html"
created: 2026-10-08
review: "unjudged"
score_reasons: ["jev unavailable: HTTP 401 after 1 attempt(s): {\"detail\":{\"error_type\":\"authentication_error\",\"message\":\"Cannot authenticate with the server. Please check your API key and try again.\"}}"]
judge_rounds: 1
---
# A lock lease without a fencing token cannot stop a paused holder from writing after its lease expired

## Core idea
A lease lock in Redis (`SET key token NX PX ttl`) expires so that a crashed holder cannot block
everyone forever. The expiry is also the hole: a holder that pauses longer than the TTL, for a
garbage-collection pause, a slow network or a stalled VM, wakes up still believing it holds the
lock while a second process has already acquired it, and both write. Releasing with a
compare-and-delete script only stops the first holder from deleting the second holder's lock; it
does not stop the first holder's late write. A fencing token closes the hole: the lock service
returns a number that increases with every acquisition, the holder sends it with each write, and
the storage rejects any write whose token is lower than the highest it has already accepted.

## Why choose / why not
- A plain lease is enough when: the lock only avoids duplicate work, such as two workers sending
  the same report, and a rare double run is harmless.
- Add a fencing token, or let the database do the claim, when: a double write corrupts data;
  `SELECT ... FOR UPDATE SKIP LOCKED` or a version column makes the database itself the arbiter.
- Don't rely on: tuning the TTL; any finite TTL can be outlived by a long enough pause.

## Interview angle
- Asked as "how do you build a distributed lock with Redis, and what can go wrong?".
- Common wrong answer: "`SET NX` with an expiry and a random value, released by a Lua
  compare-and-delete, is safe".
- Strong answer: that prevents deleting someone else's lock, not a stale holder's write;
  correctness needs a fencing token that the resource checks, or a claim made by the database.

## Related
- [[SKIP LOCKED lets several workers claim different rows of a job table without waiting]]: a
  database claim needs no separate lock service, because the row lock and the write share one
  transaction.
- [[SELECT FOR UPDATE makes competing writers queue on the row until the lock holder commits]]:
  the same arbiter idea inside one database.
- [[Concurrency MOC]]: the map entry for locks that span processes.
- Seen in: LEO-CDP/leo-customer360, customer360-backend/shared/redis_lock.py, a `SET NX EX` lease
  whose holders write to PostgreSQL without any token check (read 2026-10-08).
