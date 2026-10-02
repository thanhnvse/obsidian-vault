---
tags: [kafka, messaging, delivery-semantics, offsets, microservices, interview]
status: draft
author: claude
up: ["[[Microservices and messaging MOC]]"]
source: "https://kafka.apache.org/36/javadoc/org/apache/kafka/clients/consumer/KafkaConsumer.html"
created: 2026-09-30
score: 0.883
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Kafka auto-commit commits the offsets that poll() returned, not the records the application finished

## Core idea
In Kafka 3.6, `enable.auto.commit` is `true` by default, and the consumer then commits offsets
automatically every `auto.commit.interval.ms` (5000 ms by default). The offset it commits is the
consumer's position: the offset of the next record after the last one that `poll()` returned to
the application, whether or not the application has finished processing those records. The
`KafkaConsumer` javadoc therefore says that automatic commits give at-least-once delivery only if
the application consumes all data returned from each call to `poll()` before any subsequent
call, or before closing the consumer. In a test on Kafka 3.6.1 with `auto.commit.interval.ms=0`,
the group's committed offset reached 3 while the consumer kept polling, although none of the
three polled records had been processed. An application that hands polled records to another
thread and keeps polling therefore commits work that is not done, and a crash loses that work.

## Why choose / why not
- Keep auto-commit when: the poll loop processes every record synchronously before it polls
  again, and processing the last interval's records a second time after a crash is acceptable.
- Disable it when: records are handed to worker threads, buffered, or written in batches later;
  commit explicitly once the work is done.
- Don't use it when: the effect must not be lost, because the committed position follows what
  `poll()` returned, not what was finished.

## Interview angle
- Probed as "we run the consumer with the default settings; what delivery guarantee do we have?"
  It depends on the loop: at-least-once if each batch is processed inside the loop, and possible
  loss once processing moves to another thread.
- Common wrong answer: "auto-commit commits after my handler finishes." It commits a position,
  and the position moves when `poll()` returns records.
- Strong answer: explain the position, the condition the javadoc gives for at-least-once, and the
  asynchronous hand-off trap, then say you disable auto-commit and commit after processing.

## Related
- [[Committing Kafka offsets after processing gives at-least-once delivery, so consumers must be idempotent]]:
  that note is about choosing the order of processing and committing by hand; auto-commit makes
  the choice implicitly, based on what `poll()` returned rather than on what was processed.
- [[Microservices and messaging MOC]]: the map this note sits under, in its Kafka section next to
  the other delivery-semantics notes.
