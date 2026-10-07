---
tags: [redis, replication, durability, system-design, interview]
status: draft
author: claude
up: ["[[Caching MOC]]", "[[System design MOC]]"]
source: "https://redis.io/docs/latest/operate/oss_and_stack/management/replication/"
created: 2026-10-07
review: unjudged
---
# By default Redis replication is asynchronous, so a write the master acknowledged can be lost when a replica is promoted

## Core idea
A Redis replica applies the stream of write commands its master sends, and by default the master
acknowledges a write to the client without waiting for any replica. If the master dies after the
`OK` and before the replica has the write, and that replica is promoted, the write is gone. In the
write-up's test the replica was cut off with `docker network disconnect`: `SET` still returned `OK`,
`WAIT 1 200` returned `0`, and after the master was killed and the replica promoted with
`REPLICAOF NO ONE` the key was missing. `WAIT` and `min-replicas-to-write` shrink the window (the
master answers `NOREPLICAS` once no replica is close enough) but do not close it; Sentinel and
Cluster automate the promotion, not the guarantee.

## Why choose / why not
- Accept asynchronous replication when: Redis holds what can be rebuilt or bounded, such as a cache,
  counters, rate limits or a leaderboard, and a lost tail is tolerable (chấp nhận được).
- Keep the truth in the database when: the data is money, orders or idempotency keys; Redis then
  only decides who gets to try.
- Add `min-replicas-to-write` when: you prefer refusing writes to an unbounded loss; it limits the
  time window of loss and is best effort.
- Don't read your own write from a replica when: the user must see it; a cut-off replica kept
  answering `100` while the master held `120`.
- Don't let a master with persistence off restart empty on its own: its replicas copy the empty
  dataset, and in the write-up's test the replica ended with 0 keys.

## Interview angle
- Probed as "can an acknowledged write be lost in Redis?" or "Redis failed over during a flash sale;
  what is gone?"
- Common wrong answers: "Sentinel guarantees no data loss", "`WAIT` makes Redis strongly consistent",
  "with AOF Redis never loses data".
- Strong answer: yes, the master answers before the replica has the write; Sentinel and Cluster
  automate the promotion only; `WAIT` and `min-replicas-to-write` shrink the window; the truth
  lives in the database, whose conditional `UPDATE` stays the guard.

## Related
- [[Caching MOC]]: the Redis map; this is the durability limit to state whenever Redis holds more
  than a disposable cache.
- [[System design MOC]]: replication and failover answer "what gives way first" for the data tier.
- [[Read replicas scale reads but serve stale data while replication lags]]: the same
  asynchronous-copy trade-off in PostgreSQL; there the symptom is a stale read, here the sharper edge
  is a lost acknowledged write at promotion.
- Written up in win-interview: backend/docs/redis.md, section 2.6
