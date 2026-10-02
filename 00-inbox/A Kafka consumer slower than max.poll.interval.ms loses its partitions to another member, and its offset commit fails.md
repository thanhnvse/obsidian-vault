---
tags: [kafka, messaging, consumer-group, rebalancing, microservices, interview]
status: draft
author: claude
up: ["[[Microservices and messaging MOC]]"]
source: "https://kafka.apache.org/36/configuration/consumer-configs/"
created: 2026-09-30
score: 0.897
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A Kafka consumer slower than max.poll.interval.ms loses its partitions to another member, and its offset commit fails

## Core idea
A Kafka consumer in a group is checked for liveness in two ways: it sends periodic heartbeats,
and one that cannot send them for `session.timeout.ms` (45 s by default in Kafka 3.6) is
considered dead; separately, the application must call `poll()` again within
`max.poll.interval.ms` (300 s by default). The Kafka 3.6 consumer
configs state that if `poll()` is not called before `max.poll.interval.ms` expires, the consumer is
considered failed and the group rebalances to reassign its partitions to another member. The
records that the slow consumer received but had not committed are then delivered again to the new
owner, and the `KafkaConsumer` javadoc says the slow consumer's own offset commit fails with
`CommitFailedException`. In a test on Kafka 3.6.1, a consumer with `max.poll.interval.ms=3000`
stopped polling after receiving three records; when a second consumer joined, the second consumer
received the same three records and the first consumer's `commitSync()` threw
`CommitFailedException`.

## Why choose / why not
- Lower `max.poll.records` or speed up processing when: one batch can take close to
  `max.poll.interval.ms`, because each `poll()` then returns less work and the loop polls in time.
- Move the processing to another thread when: single records are slow; the `KafkaConsumer`
  javadoc recommends pausing the partition and keeping `poll()` going while the worker runs.
- Don't simply raise `max.poll.interval.ms`: a consumer that is really stuck then keeps its
  partitions, unprocessed, for that much longer before the group gives them to another member.

## Interview angle
- Probed as "the same messages are processed twice and the logs show `CommitFailedException`;
  why?" The batch took longer than `max.poll.interval.ms`, the group gave the partition to another
  member, and that member redid the uncommitted work.
- Common wrong answer: "heartbeats keep the consumer alive, so slow processing is fine."
  Heartbeats only prove that the process is alive; the gap between `poll()` calls is checked
  separately.
- Strong answer: name both timers and the redelivery, then the fixes in order: smaller batches,
  processing on another thread with `pause()` and `resume()`, and only then a larger interval.

## Related
- [[Committing Kafka offsets after processing gives at-least-once delivery, so consumers must be idempotent]]:
  this note names a common trigger of the redelivery that note describes: the partition moves to
  another member before the slow consumer commits.
- [[The partition count caps how many consumers in a Kafka consumer group can do work]]: that note
  explains the one-member-per-partition rule; this note shows the rule handing a partition to a new
  member when the old one stops polling.
- [[Microservices and messaging MOC]]: the map this note sits under, in its Kafka section next to
  the other consumer group notes.
