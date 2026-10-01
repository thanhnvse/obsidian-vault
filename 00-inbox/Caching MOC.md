---
tags: [moc, system-design, caching, interview]
type: moc
status: draft
author: claude
up: ["[[System design MOC]]"]
created: 2026-10-01
---
# Caching MOC

The question behind this map: *what makes a cached value wrong, and for how long?*

## Strategies
- [[Cache-aside loads data on a miss and leaves cache consistency to the application]]: the default strategy and its ordering rule
- [[Write-behind caching makes the cache the system of record until its queue is flushed]]: the fast option that trades durability

## Staleness
- [[A correctly ordered cache-aside delete still lets a slow reader write a stale value back]]: the race that survives the right order
- [[In Redis a plain SET on an existing key clears its TTL]]: how the staleness bound silently disappears

## Load
- [[A cache stampede happens when a hot key expires and many requests regenerate it at once]]: what an expiring hot key does to the database
- [[Adding nodes to a Redis Cluster cannot relieve a single hot key]]: why one key stays on one node
