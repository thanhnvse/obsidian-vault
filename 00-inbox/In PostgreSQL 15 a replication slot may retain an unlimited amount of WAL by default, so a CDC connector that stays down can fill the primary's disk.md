---
tags: [microservices, messaging, outbox, postgresql, interview]
status: draft
author: claude
up: ["[[Microservices and messaging MOC]]"]
source: "https://www.postgresql.org/docs/15/runtime-config-replication.html"
created: 2026-10-07
review: unjudged
---
# In PostgreSQL 15 a replication slot may retain an unlimited amount of WAL by default, so a CDC connector that stays down can fill the primary's disk

## Core idea
Debezium reads the outbox through logical decoding: it needs `wal_level=logical` and a replication
slot. A slot keeps the WAL its consumer has not read yet, and keeps it while the connector is down.
Debezium's documentation says its slots retain all the WAL that Debezium needs, even during outages,
and tells you to monitor them to avoid too much disk consumption. `max_slot_wal_keep_size` caps that
retention, but its default, `-1`, lets slots retain an unlimited amount of WAL. A connector that is
down over a weekend, or a slot left behind after a connector was retired, can therefore fill the
primary's disk. This is sourced from the PostgreSQL and Debezium documentation; the lab did not run it.

## Why choose / why not
- Choose CDC when: volume or latency has outgrown polling, or the platform already runs Debezium;
  it reads committed work in commit order with low latency.
- Don't choose it when: the platform does not run Debezium; a polling relay works with any SQL
  database and needs no slot, Kafka Connect or WAL to watch.
- Watch the slot, not only the connector: look for a `pg_replication_slots` row with an old
  `restart_lsn` and alert on the primary's disk; `max_slot_wal_keep_size` is the cap, and its
  default is no cap.
- Don't leave a slot behind when a connector is retired: it keeps its WAL until someone drops it.

## Interview angle
- Asked as "polling or CDC for the outbox?", or "why not Debezium everywhere?".
- Common wrong answer: "CDC is free: it just reads the log".
- Strong answer: commit order and low latency against Kafka Connect and a slot; the slot holds WAL
  while the connector is down and nothing caps it by default, so you monitor it; start with polling
  unless volume, latency or an existing Debezium setup says otherwise.

## Related
- [[A transactional outbox sends a message if and only if the database transaction commits, but its relay can send it twice]]:
  CDC is the "transaction log tailing" relay that note names; this note is its main operational
  cost.
- [[Microservices and messaging MOC]]: the map this belongs to; it is the "slow or down" half of its
  question, applied to the connector itself.

Written up in win-interview: backend/docs/idempotency-and-outbox.md, section 2.6 (and 3.1)
