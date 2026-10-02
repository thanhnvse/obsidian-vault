---
tags: [kafka, messaging, offsets, consumer-groups, microservices, interview]
status: draft
author: claude
up: ["[[Microservices and messaging MOC]]"]
source: "https://kafka.apache.org/36/configuration/consumer-configs/"
created: 2026-10-01
score: 0.903
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A new Kafka consumer group starts at the end of the log by default, so it skips every record already there

## Core idea
A Kafka consumer group resumes each partition from the offset it committed there. A group that has
never committed has no offset to resume from, and the consumer setting `auto.offset.reset` decides
where it starts instead. The Kafka 3.6 consumer configs define its values as `earliest`
(automatically reset the offset to the earliest offset), `latest` (automatically reset the offset to
the latest offset) and `none` (throw an exception to the consumer if no previous offset is found for
the group), and give `latest` as the default. A brand-new group with default settings therefore
starts at the end of each partition and receives only records produced after it was assigned the
partition; the records already in the topic are skipped, and nothing reports an error. The same
setting applies when the group's committed offset no longer exists on the server, for example
because retention deleted that data, so by default a group that falls behind retention also jumps
to the end. In a test on Kafka 3.6.1, a new group on a topic that already held three records was
positioned at offset 3 and received only the fourth record, sent after it joined.

## Why choose / why not
- Choose `earliest` when: a new service must build its state from the history in the topic, such as
  a projection or a search index, or when reprocessing after an offset is deleted is safer than
  skipping; the handler must then cope with replaying old records.
- Keep `latest` when: only events from now on matter to the service, such as live alerts or
  monitoring, and replaying the topic's history would be wrong or expensive.
- Choose `none` when: a missing offset should stop the consumer so an operator decides, instead of
  the client silently skipping or replaying.

## Interview angle
- Probed as "we deployed a new consumer for an existing topic and it never saw the old events; why?"
  The group had no committed offset, and `auto.offset.reset` defaults to `latest`.
- Common wrong answer: "a new consumer group reads the topic from the beginning." Only with
  `earliest`, or with an explicit seek to the beginning.
- Strong answer: name the setting and its default, the second trigger (a committed offset deleted
  by retention), and choose it per group by asking whether a replay or a gap is the safer failure.

## Related
- [[Committing Kafka offsets after processing gives at-least-once delivery, so consumers must be idempotent]]:
  that note covers where a group resumes when it has a committed offset; this note covers the case
  where it has none, which `auto.offset.reset` decides instead.
- [[Kafka auto-commit commits the offsets that poll() returned, not the records the application finished]]:
  another consumer default that can leave records unprocessed without any error: auto-commit when
  processing moves to another thread, `latest` for records produced before a new group's first start.
