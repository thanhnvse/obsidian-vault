---
tags: [kafka, messaging, idempotency, delivery-semantics, microservices, interview]
status: draft
author: claude
up: ["[[Microservices and messaging MOC]]"]
source: "https://kafka.apache.org/36/javadoc/org/apache/kafka/clients/producer/KafkaProducer.html"
created: 2026-10-01
score: 0.91
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Kafka's idempotent producer drops only its own retries within one session, not a second send() of the same record

## Core idea
Kafka's idempotent producer, enabled by default since Kafka 3.0 when no conflicting setting is
made, works by numbering: the broker assigns each producer an ID and deduplicates messages using
a sequence number that the producer sends with every message. A retry after a lost
acknowledgement carries the same sequence number, so the broker does not write a second copy to
the log. A second `send()` call from the application is a new message to the producer and gets
the next sequence number, so the broker writes it as a second record even when its key and value
are identical; the broker never compares content. The Kafka 3.6 `KafkaProducer` javadoc states
both limits: application-level re-sends cannot be de-duplicated, and the producer can only
guarantee idempotence for messages sent within a single session, so a service that restarts and
resends is not deduplicated either. In a test on Kafka 3.6.1, two `send()` calls of the same key
and value from one idempotent producer left two records in the partition.

## Why choose / why not
- Rely on the idempotent producer when: the duplicates to prevent are the producer's own retries
  after a timeout or a lost acknowledgement; keep `enable.idempotence`, `retries` and `acks=all`
  at their defaults and it needs no code.
- Don't rely on it when: duplicates can arise above the producer, such as an HTTP client retrying
  the request that triggers the `send()`, or a service that restarts and replays its work; put a
  business id in the record and make the consumer deduplicate on it.

## Interview angle
- Probed as "we turned on the idempotent producer; can the topic still hold duplicates?" Yes:
  from application resends and from a new producer session, because only retries inside one
  session are deduplicated.
- Common wrong answer: "the broker drops a record with the same key and value", or "the
  idempotent producer gives end-to-end exactly-once". The broker compares producer ID and
  sequence numbers, never the payload.
- Strong answer: producer ID plus per-partition sequence numbers, what that catches (the
  producer's retries), what it misses (a second `send()`, a restart), and the idempotent consumer
  that covers the rest.

## Related
- [[Kafka exactly-once covers Kafka-to-Kafka processing, not external side effects]]: that note
  names the idempotent producer as one building block of exactly-once; this note pins down how
  narrow that block is, which is why duplicates created by the application stay possible even
  with exactly-once configured.
- [[Committing Kafka offsets after processing gives at-least-once delivery, so consumers must be idempotent]]:
  the duplicates the producer cannot catch reach the consumer like any redelivery, so the
  idempotent handler described there is what absorbs them.
