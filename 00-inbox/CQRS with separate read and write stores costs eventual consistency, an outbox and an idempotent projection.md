---
tags: [system-design, interview, cqrs, architecture, consistency]
status: draft
author: claude
up: ["[[System design MOC]]"]
source: "https://learn.microsoft.com/en-us/azure/architecture/patterns/cqrs"
created: 2026-10-01
score: 0.867
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# CQRS with separate read and write stores costs eventual consistency, an outbox and an idempotent projection

## Core idea
In CQRS, commands that update data and queries that read it use separate models. When the read model gets its own store, that store is a second copy of the data that the write side must keep in step, and the main costs of the pattern come from keeping it in step asynchronously. The write side usually publishes an event for each change, which the read model uses to refresh its data. Because a message broker and a database usually cannot join one distributed transaction, Microsoft's CQRS guidance recommends the Transactional Outbox pattern to persist the change and its event atomically, and an idempotent read-model consumer to tolerate duplicate delivery. Until the projection has applied an event, the read store does not show the most recent change, so readers see eventually consistent data.

## Why choose / why not
- Choose a separate read store when: read and write shapes really differ or must scale independently, such as a catalogue read far more often than it is written; accept a visible lag between a save and the list that shows it.
- Choose separate models in one database when: you want query-shaped DTOs and a clean command model but no projection to run; the cost is two code paths, not a pipeline.
- Don't use CQRS when: the domain or business rules are simple and CRUD-style screens are enough; Microsoft lists both as cases where the pattern might not be suitable.

## Interview angle
- Probed as "would you use CQRS for this admin screen?", or "the user saves and the list still shows the old value; why?"
- Common wrong answer: "CQRS and event sourcing for scalability", with nothing on the lag, the outbox or duplicate events.
- Strong answer: start with separate models in one database; move to a separate read store only for a measured read/write asymmetry, and name the bill: an outbox, an idempotent projection, and a stale-read window the UI is designed around.

## Related
- [[Denormalise a read path only with a mechanism that keeps the copies consistent]]: a CQRS read store is a denormalised copy at service scale; there a trigger keeps the copy in step inside one transaction, here an outbox and a projection do it asynchronously.
- [[Read replicas scale reads but serve stale data while replication lags]]: both scale reads with a copy that lags; a replica copies the same schema through the database, while a CQRS read store holds a different shape that the application builds.
- [[A transactional outbox sends a message if and only if the database transaction commits, but its relay can send it twice]]: that note explains the outbox mechanism itself; this one is the decision to take on an outbox at all, as one line of the bill for a separate read store.
