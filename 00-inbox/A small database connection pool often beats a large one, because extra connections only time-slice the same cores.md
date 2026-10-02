---
tags: [system-design, interview, database, connection-pool, scaling]
status: draft
author: claude
up: ["[[System design MOC]]"]
source: "https://github.com/brettwooldridge/HikariCP/wiki/About-Pool-Sizing"
created: 2026-10-01
score: 0.853
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A small database connection pool often beats a large one, because extra connections only time-slice the same cores

## Core idea
A CPU core executes one thread at a time; when a database server runs more active queries than it has cores, the operating system time-slices between them, which adds context switches without adding work done. The HikariCP pool-sizing guide uses this to argue that once the threads exceed the CPU cores, adding more makes execution slower, not faster; disk and network waits are what let a database usefully run somewhat more connections than cores. As a starting point to test and adjust, it quotes a formula from the PostgreSQL project, connections = ((core_count * 2) + effective_spindle_count). It cites an Oracle Real-World Performance demonstration in which reducing the pool size alone cut application response times from about 100 ms to about 2 ms (the guide's example). Its axiom is a small pool saturated with threads waiting for connections: requests queue cheaply in the application instead of contending inside the database.

## Why choose / why not
- Start from the formula and load-test when: sizing the pool for a service against its database; a 4-core server with one disk gives 9 as the first guess (example numbers).
- Divide that budget across instances when: many instances share one database; each instance's pool is the database's budget divided by the instance count, not the instance's request-thread count.
- Don't enlarge the pool to cure connection waits when: database CPU or locks are already saturated; more connections deepen the contention, so find the slow queries or the code that holds connections first.

## Interview angle
- Probed as "how many connections should the pool have?", or "requests wait for a connection; should we double the pool?"
- Common wrong answer: "bigger pool, more throughput", or a pool sized to the number of request threads.
- Strong answer: derive the size from the database's cores and disks, test it under load, share it across instances, and treat threads waiting on a small pool as the intended queue.

## Related
- [[Scaling out app instances multiplies database connections, so instances times pool size must stay under max_connections]]: that note is the hard ceiling set by max_connections; this one argues that the useful pool size sits far below that ceiling.
- [[REQUIRES_NEW keeps the outer connection while it borrows a second one from the pool]]: a small pool leaves little slack, so a code path that holds two connections per request is the first to exhaust it.
