---
tags: [system-design, caching, consistency, interview]
status: draft
author: claude
up: ["[[Caching MOC]]"]
source: "https://learn.microsoft.com/en-us/azure/architecture/patterns/cache-aside"
created: 2026-10-01
score: 0.875
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 2
---
# A correctly ordered cache-aside delete still lets a slow reader write a stale value back

## Core idea
In cache-aside the writer updates the data store and then deletes the cached key, which is the order Microsoft's Cache-Aside guidance prescribes. That order still leaves a race: a reader that missed the cache just before the write may already hold the old row, and if its cache write lands after the writer's delete, the old row is cached again. At that point nothing in the cache tells the writer or later readers that the entry is old, so it stays until it expires. The guidance itself states that cache-aside does not guarantee consistency between the data store and the cache, which makes the expiration time the real bound on how long such a stale entry survives.

## Why choose / why not
- Accept the TTL bound when: a few seconds of staleness is harmless and the TTL is short; it costs nothing extra.
- Add versioned compare-and-set writes when: a stale value is a correctness bug (prices, permissions); the cost is a version on every row and a conditional cache write path.

## Interview angle
- Probed as "why isn't delete-after-commit enough?"; deriving the slow-reader race separates understanding from a memorised ordering rule.
- Common wrong answer: "we invalidate after commit, so the cache is always consistent."
- Strong answer: draw the interleaving, name the TTL as the bound, then give versioned writes or write-through as the fixes and their cost.

## Related
- [[Cache-aside loads data on a miss and leaves cache consistency to the application]]: that note explains the ordering rule and the TTL bound; this one shows the race that survives the correct order.
- [[In Redis a plain SET on an existing key clears its TTL]]: the TTL is the only bound here, and a plain SET can silently remove it.
