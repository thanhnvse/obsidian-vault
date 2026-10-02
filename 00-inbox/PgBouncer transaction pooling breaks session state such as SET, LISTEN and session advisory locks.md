---
tags: [database, postgresql, pooling, interview]
status: draft
author: claude
up: ["[[Database MOC]]"]
source: "https://www.pgbouncer.org/features.html"
created: 2026-10-01
score: 0.877
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# PgBouncer transaction pooling breaks session state such as SET, LISTEN and session advisory locks

## Core idea
In transaction pooling mode, PgBouncer assigns a server connection to a client only for the duration of a transaction and puts it back into the pool when the transaction is over. Session-level state therefore does not follow the client to its next transaction: PgBouncer 1.26 lists SET/RESET, LISTEN, WITH HOLD cursors, SQL PREPARE/DEALLOCATE and session-level advisory locks as incompatible with transaction pooling. PgBouncer says this mode breaks client expectations of the server by design and needs the application's cooperation to avoid the features that do not work. In return, many application instances can share a small number of database connections.

## Why choose / why not
- Choose transaction pooling when: many instances run short, stateless transactions; it multiplexes them onto few server connections.
- Choose session pooling, or no pooler, when: the code relies on LISTEN, session advisory locks or session SET; those break silently in transaction mode.

## Interview angle
- Probed as "our advisory locks stopped working after we added PgBouncer; why?"
- Common wrong answer: "PgBouncer is transparent to the application."
- Strong answer: name the pooling mode, explain that the server connection changes between transactions, then list what that breaks.

## Related
- [[Scaling out app instances multiplies database connections, so instances times pool size must stay under max_connections]]: that note explains why teams put PgBouncer in front of PostgreSQL at all, because many app instances times their pool size would exceed max_connections; this note is the price of that fix, the session features transaction pooling gives up.
