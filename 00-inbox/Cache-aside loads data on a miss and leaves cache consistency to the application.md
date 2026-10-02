---
tags: [system-design, interview, caching, redis, consistency]
status: draft
author: claude
source: "https://learn.microsoft.com/en-us/azure/architecture/patterns/cache-aside"
created: 2026-09-30
score: 0.81
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Cache-aside loads data on a miss and leaves cache consistency to the application

## Core idea
With cache-aside, the application reads the cache first; on a miss it reads the item from the data
store, adds it to the cache and returns it. When the application updates data, it writes the change
to the data store and then invalidates the cached item, so the next read reloads the new value. The
order matters: removing the cached item before updating the store leaves a window in which a reader
can load the old value back into the cache. Cache-aside does not guarantee consistency: a change
made to the data store by another process is not seen until the item reloads, so an expiration time
is what bounds how stale an item can get.

## Why choose / why not
- Choose cache-aside when: data is read far more often than it is written, tolerates some
  staleness, and the cache has no built-in read-through or write-through; only requested data ends
  up in the cache.
- Don't choose it when: most requests miss anyway, because checking and filling the cache then costs
  more than it saves.
- Use write-through instead when: read-heavy paths need read-after-write freshness, because
  write-through updates the data store and the cache in the same write.

## Interview angle
- Probed as "walk me through your cache on an update"; the expected sequence is update the database,
  then delete the key, and the order is the point.
- Common wrong answer: "delete the key, then update the database", which lets a concurrent reader
  put the old value back.
- Strong answer: give the sequence, the TTL as a safety net for writes the application never sees,
  and the point at which you would switch to write-through.

## Related
- [[Read replicas scale reads but serve stale data while replication lags]]: both are read-heavy
  levers that buy capacity with staleness; if the miss path reads from a lagging replica, it can put
  the old value back in the cache.
- [[System design MOC]]: the first caching note in the System design cluster.
