---
tags: [system-design, interview, messaging, idempotency, reliability]
status: draft
author: claude
up: ["[[Microservices and messaging MOC]]"]
source: "https://learn.microsoft.com/en-us/azure/architecture/patterns/idempotent-consumer"
created: 2026-10-01
score: 0.887
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# An idempotent consumer records a stable message key in the same transaction as its effect, under a unique constraint

## Core idea
Most brokers, including Azure Service Bus, Apache Kafka and RabbitMQ, deliver at least once, and Microsoft's Idempotent Consumer pattern calls exactly-once delivery across a distributed system impractical, so the consumer has to make a second delivery harmless. It records a deduplication key for each message it processes and skips any message whose key is already recorded. Three details decide whether that works. The key must stay the same across redeliveries: a producer-assigned message ID or a business idempotency key, never a transport-level identifier that the broker regenerates on redelivery. The key and the business change must commit in the same transaction, because recording the key in a separate step leaves a crash window in which the effect is applied but the key is not, so the next redelivery runs it again. And a uniqueness constraint on the key must arbitrate, because two competing consumers can both pass an existence check before either one commits.

## Why choose / why not
- Choose a deduplication table (an inbox) when: the effect is not naturally repeatable, such as an increment, a charge or a notification.
- Prefer a naturally idempotent write when: the message carries absolute state; an upsert keyed on a business ID gives the same result twice with no bookkeeping.
- Keep the markers longer than the broker can redeliver when: operators resubmit messages from a dead-letter queue; deleting markers early reopens the window for duplicates.
- Don't rely on broker duplicate detection alone when: duplicates come from redelivery; Service Bus duplicate detection works on the send side and within a bounded window.

## Interview angle
- Probed as "the consumer committed and then crashed before the ack; what happens on redelivery?"
- Common wrong answer: "we check whether the ID exists, then insert", keyed on the broker's delivery ID and with no unique constraint.
- Strong answer: a producer-assigned key, marker and effect in one transaction, a unique constraint that turns the concurrent duplicate into a constraint violation the consumer acknowledges and skips, and markers kept longer than any redelivery or redrive.

## Related
- [[Committing Kafka offsets after processing gives at-least-once delivery, so consumers must be idempotent]]: that note says why a Kafka consumer must be idempotent and covers the naturally idempotent upsert; this one is how to build a consumer whose effect is not naturally idempotent.
- [[Kafka exactly-once covers Kafka-to-Kafka processing, not external side effects]]: the reason the consumer, not the broker, has to deduplicate an effect that lands in a database.
- [[A transactional outbox sends a message if and only if the database transaction commits, but its relay can send it twice]]: that note ends with "every consumer must be idempotent"; this one is how to build that consumer, and the outbox's own message ID is a natural stable key for it.
