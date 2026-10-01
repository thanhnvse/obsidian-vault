---
tags: [system-design, caching, redis, interview]
status: draft
author: claude
up: ["[[Caching MOC]]"]
source: "https://redis.io/docs/latest/commands/expire/"
created: 2026-10-01
score: 0.903
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# In Redis a plain SET on an existing key clears its TTL

## Core idea
A Redis key's timeout is cleared only by commands that delete or overwrite the key's contents, including DEL, SET, GETSET and the *STORE commands. Commands that alter the value without replacing it, such as INCR, LPUSH or HSET, leave the timeout untouched. Cache code that refreshes an entry with a plain SET therefore removes its expiration, and the EXPIRE documentation's own example shows TTL returning -1 after such a SET. Under cache-aside, where the expiration is the bound on staleness, one missed invalidation then becomes a value that never expires. Setting the value and the expiry in one command, SET key value EX seconds, avoids the gap.

## Why choose / why not
- Use SET key value EX seconds when: writing any cache entry; value and TTL are set atomically and cannot drift apart.
- Keep keys without a TTL only when: the data is primed deliberately and replaced on purpose; anything else accumulates and goes stale.

## Interview angle
- Probed as a debugging question: "a cached value never refreshes; how do you find out why?"; run TTL on the key and look for -1.
- Common wrong answer: "EXPIRE once at creation is enough."
- Strong answer: name the overwrite rule, then the atomic SET ... EX fix.

## Related
- [[Cache-aside loads data on a miss and leaves cache consistency to the application]]: there the TTL bounds staleness; this note shows how a refresh silently removes that bound.
