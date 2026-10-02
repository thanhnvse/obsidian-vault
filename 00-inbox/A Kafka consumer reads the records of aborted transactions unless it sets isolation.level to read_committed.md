---
tags: [kafka, messaging, transactions, delivery-semantics, microservices, interview]
status: draft
author: claude
up: ["[[Microservices and messaging MOC]]"]
source: "https://kafka.apache.org/36/configuration/consumer-configs/"
created: 2026-10-01
score: 0.91
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A Kafka consumer reads the records of aborted transactions unless it sets isolation.level to read_committed

## Core idea
A Kafka transaction does not keep its records out of the log until it commits: the records are
appended to the partition as they are sent, and the transaction coordinator later writes a commit
or abort marker into each partition it touched. An aborted transaction's records therefore stay in
the log, and the consumer setting `isolation.level` decides whether a reader sees them. The Kafka
3.6 consumer configs say that with `read_uncommitted`, which is the default, `consumer.poll()`
returns all messages, even transactional messages which have been aborted, while with
`read_committed` it returns only transactional messages which have been committed, and
non-transactional messages in either mode. A `read_committed` consumer also returns messages only
up to the last stable offset, so records behind a transaction that is still open are withheld until
it completes. A transactional producer therefore protects only the readers that opt in: the
`KafkaProducer` javadoc says that for transactional guarantees to be realized end to end, the
consumers must be configured to read only committed messages. In a test on Kafka 3.6.1, a consumer
with default settings returned the two records of an aborted transaction and the committed one,
while a `read_committed` consumer returned only the committed record.

## Why choose / why not
- Set `read_committed` when: the topic is written by a transactional producer, such as a
  consume-transform-produce service or a Kafka Streams application with exactly-once, and acting on
  output that was later aborted would be wrong. Every downstream reader needs it, not only the next
  service.
- Leave the default when: no producer of the topic uses transactions; non-transactional records
  are returned the same way in both modes, so the setting changes nothing.
- Expect the cost: a `read_committed` consumer cannot read past an open transaction on a partition,
  so a long or stuck transaction holds back every later record there, including non-transactional
  ones.

## Interview angle
- Probed as "the processor runs Kafka transactions and aborted a batch; can a downstream service
  still act on it?" Yes, if that service reads with the default `isolation.level`.
- Common wrong answer: "an aborted transaction's records are deleted, or never written." They stay
  in the log next to an abort marker; only the reader's setting hides them.
- Strong answer: exactly-once needs both ends, a transactional producer and `read_committed` on
  every reader, and name the last stable offset as the price a reader pays for it.

## Related
- [[Kafka exactly-once covers Kafka-to-Kafka processing, not external side effects]]: that note
  lists `read_committed` readers as one of the three parts of exactly-once; this note shows what a
  reader without it sees, the quiet way the guarantee breaks downstream.
- [[A new Kafka producer with the same transactional.id fences the old instance and aborts its open transaction]]:
  fencing aborts the zombie's open transaction, and that abort protects a downstream service only
  if the service reads with `read_committed`.
