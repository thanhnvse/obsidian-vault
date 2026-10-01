---
tags: [system-design, database, replication, interview]
status: draft
author: claude
up: ["[[System design MOC]]", "[[Database MOC]]"]
source: "https://www.postgresql.org/docs/18/warm-standby.html"
created: 2026-10-01
score: 0.888
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Read-your-writes on an asynchronous PostgreSQL replica needs recency routing, WAL-position routing or remote_apply

## Core idea
PostgreSQL streaming replication is asynchronous by default, so there is a small delay between committing a transaction on the primary and the change becoming visible on a standby. A user who saves a change and is then served by a replica can read the old value. Three fixes exist, cheapest first: send that user's reads to the primary for a short window after a write; remember the commit's WAL position and read only from a replica whose pg_last_wal_replay_lsn() has reached it; or set synchronous_commit to remote_apply, which makes each commit wait until the current synchronous standbys report that they have replayed the transaction. The PostgreSQL 18 documentation says remote_apply allows load balancing with causal consistency in simple cases.

## Why choose / why not
- Route recent writers to the primary when: occasional staleness for other users is fine; it is cheap, but the window length is a guess.
- Route by WAL position when: correctness matters per request; it is exact but needs the commit position carried from write to read.
- Use remote_apply when: every read anywhere must see committed writes; every commit then waits for the slowest synchronous standby to replay it.

## Interview angle
- Probed as "a user saves their address and sees the old one; fix it."
- Common wrong answer: "make replication faster"; asynchronous lag is never zero.
- Strong answer: name the anomaly (read-your-writes), then pick a fix by its cost.

## Related
- [[Read replicas scale reads but serve stale data while replication lags]]: that note states the problem; this one gives the fixes and their costs.
