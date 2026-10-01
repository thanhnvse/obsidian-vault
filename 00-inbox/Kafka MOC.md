---
tags: [moc, kafka, messaging, interview]
type: moc
status: draft
author: claude
up: ["[[Microservices and messaging MOC]]"]
created: 2026-10-01
---
# Kafka MOC

The question behind this map: *what does Kafka guarantee about order, delivery and duplicates, and where do those guarantees stop?*

## Ordering and parallelism
- [[Kafka orders records only within a partition, and the record key chooses the partition]]: ordering, and why the record key decides it
- [[The partition count caps how many consumers in a Kafka consumer group can do work]]: the ceiling on consumer parallelism

## Consumer groups and offsets
- [[A new Kafka consumer group starts at the end of the log by default, so it skips every record already there]]: where a group with no committed offset starts reading
- [[Kafka auto-commit commits the offsets that poll() returned, not the records the application finished]]: why the default commit is not "after processing"
- [[Committing Kafka offsets after processing gives at-least-once delivery, so consumers must be idempotent]]: the delivery guarantee most services run on
- [[A Kafka consumer slower than max.poll.interval.ms loses its partitions to another member, and its offset commit fails]]: what slow processing does to the group
- [[The default Kafka rebalance revokes every partition, while cooperative rebalancing revokes only the ones that move]]: what a rebalance costs and how to shrink it

## Duplicates and transactions
- [[Kafka's idempotent producer drops only its own retries within one session, not a second send() of the same record]]: the duplicates the idempotent producer does not remove
- [[A new Kafka producer with the same transactional.id fences the old instance and aborts its open transaction]]: how a zombie producer is stopped
- [[A Kafka consumer reads the records of aborted transactions unless it sets isolation.level to read_committed]]: the consumer half of a transaction
- [[Kafka exactly-once covers Kafka-to-Kafka processing, not external side effects]]: where exactly-once stops
- [[An idempotent consumer records a stable message key in the same transaction as its effect, under a unique constraint]]: the consumer-side fix that at-least-once delivery requires

## Operating
- [[Rolling back a Kafka producer leaves its new-format events in the topic for the old consumers]]: why a producer rollback does not take back the data
