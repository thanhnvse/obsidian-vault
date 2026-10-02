---
tags: [kafka, messaging, transactions, delivery-semantics, microservices, interview]
status: draft
author: claude
up: ["[[Microservices and messaging MOC]]"]
source: "https://kafka.apache.org/36/javadoc/org/apache/kafka/clients/producer/KafkaProducer.html"
created: 2026-09-30
score: 0.906
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A new Kafka producer with the same transactional.id fences the old instance and aborts its open transaction

## Core idea
A Kafka transactional producer is identified by its `transactional.id`, which the `KafkaProducer`
javadoc says enables transaction recovery across multiple sessions of a single producer instance.
When a new instance calls `initTransactions()`, Kafka first completes any transaction that a
previous instance with the same `transactional.id` started, and aborts it if that instance failed
with the transaction still in progress. From then on the old instance is fenced: if it is still
running, for example after a long pause, its `commitTransaction()` throws
`ProducerFencedException`, so a "zombie" cannot commit output next to its successor. Consumers
with `isolation.level=read_committed` never see the records of the aborted transaction. In a test
on Kafka 3.6.1, a producer that had sent a record in an open transaction got
`ProducerFencedException` on commit after a second producer with the same `transactional.id`
called `initTransactions()`, and a `read_committed` reader saw only the successor's record.

## Why choose / why not
- Choose a stable `transactional.id` per unit of work when: a consume-transform-produce step must
  not write duplicate output while an old instance may still be alive. The javadoc says it is
  typically derived from the shard identifier and must be unique to each producer instance.
- Don't derive it from something that changes on restart, such as a random id or the host name of
  a rescheduled pod: the new instance then gets a new id and fences nothing.
- Don't expect fencing to cover side effects: a database write or HTTP call the zombie already made
  stays done; only its Kafka records and offsets are discarded.

## Interview angle
- Probed as "an instance paused for a minute, its work was given to another instance, then it woke
  up and tried to commit; what stops the duplicate output?" Fencing by `transactional.id`.
- Common wrong answer: "the old instance lost its partitions, so its writes are rejected." Losing
  partitions in the consumer group does not stop its producer; the fencing does.
- Strong answer: `transactional.id`, `initTransactions()` aborting the open transaction,
  `ProducerFencedException` on the old instance, and `read_committed` readers.

## Related
- [[Kafka exactly-once covers Kafka-to-Kafka processing, not external side effects]]: that note
  names transactions as the basis of exactly-once; fencing is the part that keeps a zombie
  instance from breaking it, and like the rest of the transaction it does nothing for database
  writes.
- [[Microservices and messaging MOC]]: the map this note sits under, in its Kafka section after
  the exactly-once note.
