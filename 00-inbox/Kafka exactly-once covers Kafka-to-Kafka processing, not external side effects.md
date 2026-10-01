---
tags: [kafka, messaging, delivery-semantics, transactions, microservices, interview]
status: draft
author: claude
source: "https://kafka.apache.org/43/design/design/"
created: 2026-09-30
score: 0.94
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 2
---
# Kafka exactly-once covers Kafka-to-Kafka processing, not external side effects

## Core idea
Kafka's exactly-once rests on two producer features: the idempotent producer, whose retries do
not create duplicate entries in the log, and transactions, which write to several partitions
atomically. In a consume-process-produce flow the consumer's offsets are stored in an internal
Kafka topic, so the producer can commit them in the same transaction as the output records,
and consumers of the output that use `isolation.level=read_committed` see only committed
transactions. The Kafka 4.3 design documentation states that this gives exactly-once delivery
when reading, processing and writing data on Kafka topics, and that exactly-once for other
destination systems generally requires cooperation from those systems. A database write or
HTTP call made during processing is not part of the Kafka transaction, so it can run again
when the transaction aborts and the records are processed a second time.

## Why choose / why not
- Choose Kafka transactions when: a service reads from topics and writes its results only to
  other topics, and a duplicate output record would be wrong. Kafka Streams is the simplest way
  to get this; with the plain clients, set `transactional.id` on the producer and
  `read_committed` with auto-commit disabled on the consumer.
- Don't expect them to cover a database write or a call to another service: store the consumed
  offset in the same database transaction as the output, or make the effect idempotent.

## Interview angle
- Probed as "does Kafka give you exactly-once?" Yes for read-process-write inside Kafka; for
  every effect outside Kafka the guarantee is at-least-once plus idempotence.
- Common wrong answer: "`enable.idempotence=true` gives end-to-end exactly-once." It only stops
  producer retries from writing duplicates into the log; a consumer can still process a record
  twice.
- Strong answer: name all three parts (idempotent producer, offsets committed inside the
  transaction, `read_committed` downstream), then say where the guarantee ends.

## Related
- [[Committing Kafka offsets after processing gives at-least-once delivery, so consumers must be idempotent]]:
  this note is related because it explains at-least-once, the guarantee a consumer has without
  transactions. Transactions raise that guarantee to exactly-once only for records and offsets
  inside Kafka, so a database write or HTTP call still needs the idempotent handler that the
  linked note describes.
