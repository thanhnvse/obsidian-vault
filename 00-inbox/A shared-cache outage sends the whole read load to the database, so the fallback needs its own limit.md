---
tags: [system-design, interview, caching, redis, resilience]
status: draft
author: claude
up: ["[[Caching MOC]]"]
source: "https://learn.microsoft.com/en-us/azure/architecture/best-practices/caching"
created: 2026-10-01
score: 0.909
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A shared-cache outage sends the whole read load to the database, so the fallback needs its own limit

## Core idea
An application should keep working when its shared cache, such as Redis, is unavailable, and Microsoft's caching guidance says it should not become unresponsive while waiting for the cache service to resume. The obvious fallback, reading every request from the original data store, hands the database the full read load that the cache was absorbing; the guidance warns that the store can then be swamped with requests, resulting in timeouts and failed connections. The AWS Builders' Library describes the same effect: an extended cache outage causes an atypical traffic spike to the downstream service, leading to throttling or brownout. Both recommend a fallback that does not rely on the database's spare capacity: AWS combines the external cache with an in-memory cache to fall back on, or uses load shedding to cap the rate of requests sent downstream, and Azure pairs a local private cache in each instance with the Circuit Breaker pattern on the shared cache.

## Why choose / why not
- Add a small local cache as the fallback when: a few hot keys carry most reads; each instance keeps serving them while the shared cache recovers, at the cost of instances briefly disagreeing.
- Shed load or cap the fallback rate when: the database cannot serve the uncached read rate; some requests fail fast instead of all of them timing out.
- Put a short timeout and a circuit breaker on every cache call when: the cache sits on the request path; a slow cache otherwise holds request threads as badly as a dead one.
- Skip the extra layers when: a load test with caching disabled shows the database can carry the full read load; then the cache is only a latency optimisation.

## Interview angle
- Probed as "what happens if Redis goes down?"
- Common wrong answer: "we fall back to the database", with no check that the database can take the full read load.
- Strong answer: timeouts and a breaker on the cache client, a local fallback cache or a capped fallback rate, and a load test with caching disabled to prove the safeguards work, which AWS describes doing.

## Related
- [[Cache-aside loads data on a miss and leaves cache consistency to the application]]: cache-aside's miss path reads the database, so a cache outage turns every read into that miss; this note is about surviving it.
- [[A cache stampede happens when a hot key expires and many requests regenerate it at once]]: a stampede overloads the database for one key; a cache outage does it for every key at once, so the same "do not let every request through" thinking applies.
