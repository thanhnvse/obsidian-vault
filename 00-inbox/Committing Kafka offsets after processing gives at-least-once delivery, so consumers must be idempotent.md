---
tags: [kafka, messaging, delivery-semantics, idempotency, microservices, interview]
status: draft
author: claude
source: "https://kafka.apache.org/43/javadoc/org/apache/kafka/clients/consumer/KafkaConsumer.html"
created: 2026-09-30
score: 0.89
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Committing Kafka offsets after processing gives at-least-once delivery, so consumers must be idempotent

## Core idea
A Kafka consumer that processes records first and commits their offsets afterwards can crash
in the gap between the two. The consumer that takes over the partition resumes from the last
committed offset and processes those records again, which the Kafka 4.3 documentation calls
"at-least-once" delivery. Since redelivery is part of that contract, the processing must be
idempotent, so that handling a record twice leaves the same result as handling it once. The
Kafka design documentation's
example is an update keyed by a primary key, where receiving the same message twice just
overwrites a record with another copy of itself.

## Why choose / why not
- Choose commit-after-processing with an idempotent handler when: the effect, such as a
  database write, must not be lost. Make the write an upsert on a business key, or store the
  consumed offset in the same database transaction as the output.
- Don't commit before processing unless: losing a record on a crash is acceptable, such as
  sampled metrics. That order avoids duplicates by skipping records instead, which is
  at-most-once.

## Interview angle
- Probed as "where do duplicate messages come from?" Name the gap between processing and
  commit: a crash there, or the partition moving to another consumer before the commit, and
  the records are processed again from the last committed offset.
- Common wrong answer: "we commit manually, so every message is processed exactly once."
  Manual commit decides which failure you get, not whether one exists.
- Strong answer: at-least-once plus an idempotent handler, and the business key that makes
  the handler idempotent.

## Related
- [[The partition count caps how many consumers in a Kafka consumer group can do work]]: a
  partition moving to another consumer of the group is the moment uncommitted work is redone,
  so the two notes describe the same group mechanics from two sides.
- [[A synchronous service call couples the caller to the callee's availability and latency]]:
  that note names duplicates as the price of messaging; this note shows where Kafka produces
  them and how the consumer absorbs them.
