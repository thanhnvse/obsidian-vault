---
tags: [kafka, messaging, ordering, microservices, interview]
status: draft
author: claude
source: "https://kafka.apache.org/43/getting-started/introduction/"
created: 2026-09-30
score: 0.88
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Kafka orders records only within a partition, and the record key chooses the partition

## Core idea
A Kafka topic is split into partitions, and each new record is appended to one of them. The
Kafka 4.3 documentation guarantees that a consumer of a given topic-partition reads that
partition's records in exactly the order they were written, and the Kafka 2.0 introduction
states the other half: there is no total order between different partitions of a topic. When the producer sets a key but no explicit
partition, the partition is chosen from a hash of the key, so records with the same key, such
as one customer ID, land in the same partition and keep their relative order. Ordering in
Kafka is therefore per key, not per topic.

## Why choose / why not
- Key by the entity ID when: the events of one entity must be applied in order, such as the
  state changes of one order; per-key ordering then holds while the topic keeps many
  partitions.
- Don't rely on order across keys: records with different keys can sit in different
  partitions and be consumed in any relative order. A true total order needs a
  single-partition topic, which limits each consumer group to one consumer.
- Don't add partitions to a keyed topic casually: with `hash(key) % partitions` the mapping
  changes, so a key's new records can land in a different partition than its old ones, and
  Kafka does not move existing data. Size the partition count up front.

## Interview angle
- Probed as "how do you keep events in order in Kafka?" The answer is the key, because the key
  picks the partition and only a partition is ordered.
- Common wrong answer: "a Kafka topic is ordered." Only each partition is.
- Strong answer: name the key you would choose and why, then the partition-count trap.

## Related
- [[Microservices and messaging MOC]]: the entry point for these interview topics; this note
  belongs to its "Microservices and messaging" section, where ordering is the first Kafka
  guarantee to pin down.
