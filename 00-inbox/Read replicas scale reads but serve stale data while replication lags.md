---
tags: [system-design, interview, database, replication, scaling]
status: draft
author: claude
source: "https://www.postgresql.org/docs/current/warm-standby.html"
created: 2026-09-30
score: 0.893
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Read replicas scale reads but serve stale data while replication lags

## Core idea
A read replica is a read-only copy of the primary database; routing queries to it takes read load
off the primary, which Amazon RDS documents as scaling beyond one DB instance for read-heavy
workloads. The price is freshness, because the copy is updated asynchronously: in PostgreSQL 18,
streaming replication is asynchronous by default, with a small delay between a commit on the
primary and the change becoming visible on the standby. During that delay, a read sent to the
replica right after a write returns the old value.

## Why choose / why not
- Choose read replicas when: reads dominate, the primary is busy serving queries, and those reads
  can tolerate a short delay, such as catalogue pages, search results or reports.
- Send the read to the primary when: the user must see their own write at once (read-your-writes),
  such as the page shown right after "save"; a replica may still show the old value.
- Make commits wait for the replica only when: one standby must never serve a stale read; in
  PostgreSQL 18 `synchronous_commit = remote_apply` does this, at the cost of much larger commit
  delays.

## Interview angle
- Probed as "we added replicas and now users see their update vanish after saving"; the cause is
  replication lag, not a caching bug.
- Common wrong answer: "a replica is a copy, so every read returns the latest data."
- Strong answer: name the lag, then the routing rule (read-your-writes paths go to the primary,
  the rest may use replicas), and say you watch lag through `pg_stat_replication`.

## Related
- [[System design MOC]]: this note opens the System design cluster, whose question is which
  load dominates; replicas are the first answer to read load, and lag is what gives way first.
