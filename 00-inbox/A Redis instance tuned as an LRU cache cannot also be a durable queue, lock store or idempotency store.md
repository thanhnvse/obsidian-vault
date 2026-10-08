---
tags: [redis, caching, messaging, durability, interview]
status: draft
author: claude
up: ["[[Caching MOC]]"]
source: "https://redis.io/docs/latest/develop/reference/eviction/"
created: 2026-10-08
review: "unjudged"
score_reasons: ["jev unavailable: HTTP 401 after 1 attempt(s): {\"detail\":{\"error_type\":\"authentication_error\",\"message\":\"Cannot authenticate with the server. Please check your API key and try again.\"}}"]
judge_rounds: 1
---
# A Redis instance tuned as an LRU cache cannot also be a durable queue, lock store or idempotency store

## Core idea
With `maxmemory` set and `maxmemory-policy allkeys-lru`, Redis makes room for new writes by
evicting keys chosen from all keys. For a cache that is the point. For a stream behind an
"accepted" response, a lock lease, or a set of idempotency keys, it means acknowledged data can
disappear while writes keep succeeding and nothing reports an error. Data that must survive memory
pressure needs the `noeviction` policy, under which writes fail with an error the caller can see,
or a separate Redis instance, or backpressure before memory runs out; the Redis documentation itself
suggests two separate instances when one would hold both cache data and keys that must persist. The `volatile-*` policies
evict only keys that have a TTL, which spares keys without expiry but still drops lock and
idempotency keys, because those carry a TTL.

## Why choose / why not
- Share one instance when: everything on it is a cache that can be rebuilt from the source of truth.
- Use a separate instance with `noeviction` when: it holds streams, locks or dedupe keys; size it
  and alert on memory use.
- Don't count on: a stream length cap (`MAXLEN`) as memory protection; it caps entries, not bytes.

## Interview angle
- Asked as "can we use our cache Redis as the job queue too?".
- Common wrong answer: "yes, Redis persists streams to disk".
- Strong answer: persistence (AOF or RDB) protects against restarts, not against eviction; the
  eviction policy decides whether acknowledged messages survive memory pressure.

## Related
- [[Redis evicts keys only at maxmemory, and maxmemory 0, the 64-bit default, sets no limit at all]]:
  that note is when eviction starts; this one is what it destroys when one instance plays several roles.
- [[An idempotent consumer records a stable message key in the same transaction as its effect, under a unique constraint]]:
  a dedupe store has to be as durable as the effect it protects, which an evicting cache is not.
- [[Caching MOC]]: the map entry for what a cache may and may not be trusted with.
- Seen in: LEO-CDP/leo-customer360, redis/redis.conf (`maxmemory 256mb`, `allkeys-lru`) on the
  instance that also holds the tracking stream, a lock lease and idempotency keys (read 2026-10-08).
