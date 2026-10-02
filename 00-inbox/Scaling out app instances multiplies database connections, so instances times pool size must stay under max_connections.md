---
tags: [system-design, database, scaling, interview]
status: draft
author: claude
up: ["[[System design MOC]]", "[[Database MOC]]"]
source: "https://www.postgresql.org/docs/18/runtime-config-connection.html"
created: 2026-10-01
score: 0.903
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Scaling out app instances multiplies database connections, so instances times pool size must stay under max_connections

## Core idea
PostgreSQL's max_connections determines the maximum number of concurrent connections to the server; the default is typically 100, and it can only be set at server start because PostgreSQL sizes resources such as shared memory from it. Every application instance brings its own connection pool, so the database sees instances times pool size connections. Scaling a service from 10 to 30 instances with a pool of 10 each raises that from 100 to 300 (example numbers), and the extra instances arrive at peak load, exactly when the database can least absorb them. The remedies are smaller pools per instance, a maximum instance count on the autoscaler, or a connection pooler such as PgBouncer between the instances and the database.

## Why choose / why not
- Shrink each pool when: instances are many and each does modest database work; a pool sized for one big instance is wrong for thirty small ones.
- Put PgBouncer in front when: the instance count must stay elastic; it lets many clients share few server connections, at the cost of session features in transaction mode.
- Don't just raise max_connections when: the server is already busy; each connection costs memory and a backend process, and the limit only changes at restart.

## Interview angle
- Probed as "the service runs 30 replicas with a pool of 20; what happens at the next scale-out?"
- Common wrong answer: "set max_connections to 10,000."
- Strong answer: multiply instances by pool size, compare with the limit, then size pools or add a pooler.

## Related
- [[Stateless services scale out, while scaling up one machine stops at a hardware limit]]: scaling out is the goal; this note is the database cost it hides.
- [[REQUIRES_NEW keeps the outer connection while it borrows a second one from the pool]]: pool exhaustion inside one instance, the same budget seen from the application.
- [[PgBouncer transaction pooling breaks session state such as SET, LISTEN and session advisory locks]]: the usual remedy, and what it breaks.
