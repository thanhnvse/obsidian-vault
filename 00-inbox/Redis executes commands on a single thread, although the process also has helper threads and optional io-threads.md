---
tags: [redis, caching, concurrency, interview]
status: draft
author: claude
up: ["[[Caching MOC]]"]
source: "https://redis.io/docs/latest/operate/oss_and_stack/management/optimization/benchmarks/"
created: 2026-10-07
review: unjudged
---
# Redis executes commands on a single thread, although the process also has helper threads and optional io-threads

## Core idea
Redis runs commands one at a time on one thread, with an event loop serving many sockets from it,
so no two commands interleave and each single command is atomic without a lock. "Single-threaded"
covers command execution only: on `redis:8.6.7-alpine` the process also runs `bio_*` and
`jemalloc_bg_thd` helper threads, and optional I/O threads (`io-threads`, default 1, so off) read
and write sockets while commands still run on the main thread. The price is that one slow command
stalls (làm cả server đứng chờ) every client: in the write-up's test, with `busy-reply-threshold`
lowered to 100 ms, a script that never ended made another client's `PING` fail with `BUSY` until
`SCRIPT KILL` (the default threshold is 5 seconds).

## Why choose / why not
- Rely on it when: one step on one key (`INCR`, `SET ... NX`, `ZINCRBY`) must be atomic with no
  lock and no read first.
- Keep every command bounded when: other clients share the instance; use `SCAN` instead of `KEYS`
  and a bounded `LRANGE` instead of reading a huge list whole.
- Scale out with more instances, not more cores, when: one instance's CPU is the limit; the Redis
  documentation says it is not designed to benefit from multiple cores.
- Turn on `io-threads` only when: the machine has four or more cores and you have a real
  performance problem, which is what the shipped `redis.conf` suggests.
- Don't answer "fast because it is single-threaded" on its own: memory, the event loop and a
  network-bound workload carry the speed; the single thread adds no locks or thread switches and
  explains atomicity and the stall risk.

## Interview angle
- Probed as "Redis is single-threaded: how is it fast, and what does that cost you?"
- Common wrong answers: "it is fast because it is single-threaded", or "single-threaded, period".
- Strong answer: four reasons in order (memory, one executor with no locks, an event loop, a
  network-bound workload helped by pipelining), then the correction about helper and I/O threads,
  then the cost: one slow command or script stalls everyone.

## Related
- [[Caching MOC]]: the Redis map; the other Redis notes assume this execution model.
- [[Adding nodes to a Redis Cluster cannot relieve a single hot key]]: the same limit from the
  cluster side; all traffic for one key reaches one node, which can then run at full CPU.
- Written up in win-interview: backend/docs/redis.md, section 2.1
