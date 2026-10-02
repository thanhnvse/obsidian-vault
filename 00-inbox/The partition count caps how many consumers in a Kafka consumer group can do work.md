---
tags: [kafka, messaging, consumer-group, scaling, microservices, interview]
status: draft
author: claude
source: "https://kafka.apache.org/43/javadoc/org/apache/kafka/clients/consumer/KafkaConsumer.html"
created: 2026-09-30
score: 0.853
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# The partition count caps how many consumers in a Kafka consumer group can do work

## Core idea
Kafka balances the partitions of the subscribed topics across the members of a consumer
group so that each partition is assigned to exactly one consumer in the group at a time. One
consumer can own several partitions: with four partitions and two consumers, each consumer
reads two. The reverse does not hold, so a group with more consumers than partitions leaves
the extra consumers without a partition and without records. The partition count is
therefore the upper bound on how many consumers in one group process records in parallel.

## Why choose / why not
- Size the partition count for peak parallelism when: creating a topic that one group must
  drain quickly. Adding partitions later raises the cap, but on a keyed topic it also remaps
  keys to new partitions.
- Don't scale consumer instances past the partition count: the extra instances sit idle while
  still taking part in the group. Raise the throughput of each consumer, or add partitions.
- Use a share group instead when: records are independent jobs and their order does not
  matter. Share groups, production-ready since Kafka 4.2, let consumers share a partition, so
  their number can exceed the partition count.

## Interview angle
- Probed as "we doubled the consumer instances and throughput did not move; why?" Check the
  partition count first: if the group already had one consumer per partition, the new
  instances got nothing.
- Common wrong answer: "Kafka load-balances records across all consumers." A consumer group
  balances partitions, not records.
- Strong answer: a partition is the unit of both ordering and parallelism, so choosing its
  count is choosing the maximum consumer parallelism of every group.

## Related
- [[Kafka orders records only within a partition, and the record key chooses the partition]]:
  the one-consumer-per-partition rule that preserves partition order is the same rule that
  caps parallelism, and raising the cap by adding partitions remaps keys.
