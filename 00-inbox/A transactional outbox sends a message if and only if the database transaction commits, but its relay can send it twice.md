---
tags: [microservices, messaging, outbox, delivery-semantics, interview]
status: draft
author: claude
up: ["[[Microservices and messaging MOC]]"]
source: "https://microservices.io/patterns/data/transactional-outbox.html"
created: 2026-10-01
score: 0.821
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A transactional outbox sends a message if and only if the database transaction commits, but its relay can send it twice

## Core idea
A service that updates its database and also sends a message to a broker cannot make the two writes
atomic without a distributed transaction, which microservices.io rules out because the database or
the broker might not support two-phase commit. Both naive orders fail: a message sent in the middle
of the transaction goes out even if the transaction then rolls back, and a message sent after the
commit is lost if the service crashes before sending it. The Transactional Outbox pattern stores the
message in the database as part of the same transaction that updates the business entities, in an
outbox table in a relational database, and a separate message relay process sends the stored
messages to the broker. Messages are then guaranteed to be sent if and only if the database
transaction commits, and in the order the service created them. The relay might publish a message
more than once, for example when it crashes after publishing but before recording that it did, so
every consumer must be idempotent, such as by tracking the IDs of the messages it has already
processed. The relay either polls the outbox table (Polling publisher) or tails the database's
transaction log (Transaction log tailing).

## Why choose / why not
- Choose an outbox when: a service changes its own data and must announce the change, such as
  `OrderPlaced` after an order is saved, and neither a lost event nor an event for a rolled-back
  change is acceptable.
- Don't choose it when: losing a message now and then is acceptable, such as a cache eviction or a
  best-effort notification; an after-commit hook is simpler and needs no relay to run and monitor.
- Don't treat it as exactly-once: the relay can publish twice, so consumers still deduplicate on a
  message ID or a business key.

## Interview angle
- Probed as "how do you publish an event reliably when the same request also writes your
  database?", often right after the candidate says "save, then publish".
- Common wrong answer: "publish after the commit" or "publish inside the transaction". The first
  loses the event on a crash between the two writes; the second can announce a change that then
  rolls back.
- Strong answer: name the dual write, the outbox row written in the same transaction, the relay
  (polling or log tailing), and the idempotent consumer it still requires.

## Related
- [[@TransactionalEventListener moves a side effect after the commit but loses it if the process dies]]:
  that note is Spring's in-process alternative; it never announces a rolled-back change but loses
  the event on a crash, the exact gap the outbox closes by making the message part of the committed
  data.
- [[Committing Kafka offsets after processing gives at-least-once delivery, so consumers must be idempotent]]:
  the relay's duplicates reach consumers just like broker redeliveries, so the idempotent handler
  from that note is required on the consuming side.
