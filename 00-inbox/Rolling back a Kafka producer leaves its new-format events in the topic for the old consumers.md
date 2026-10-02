---
tags: [kafka, messaging, rollback, ops, interview]
status: draft
author: claude
up: ["[[Ops and cloud MOC]]"]
source: "https://kafka.apache.org/43/getting-started/introduction/"
created: 2026-10-01
score: 0.887
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Rolling back a Kafka producer leaves its new-format events in the topic for the old consumers

## Core idea
In the Apache Kafka 4.3 documentation, events are durably stored in topics, and events in a topic
can be read as often as needed: unlike traditional messaging systems, events are not deleted after
consumption. Instead, a per-topic configuration setting defines how long Kafka retains events,
after which old events are discarded. Rolling back a producer service therefore stops new events in the new format, but
it does not remove the events the new version already published. Every consumer of the topic,
including consumers that are rolled back to an older version, still reads those events until the
retention period expires, so the old code must be able to handle the new format.

## Why choose / why not
- Make event formats compatible in both directions when: producers and consumers are released and
  rolled back independently; old consumers must ignore fields they do not know, and new consumers
  must still read old events.
- Don't count on a rollback to stop downstream effects: other services may already have acted on
  the events, so correct them with compensating events instead.
- Roll out a new format consumers first when: the change adds data the old consumers would reject;
  deploy readers that tolerate it before the producer starts writing it.

## Interview angle
- Probed as "you rolled back the producer; what happens to the events it already published?"
- Common wrong answer: "the rollback cleans up the events we sent."
- Strong answer: the events stay for the topic's retention period and are read by every consumer,
  so event schemas need compatibility both ways; name deserialization errors right after a
  rollback as the symptom.

## Related
- [[Expand and contract schema changes keep the previous version runnable after a rollback]]: the
  same compatibility window, applied to an event format instead of a database schema.
- [[kubectl rollout undo rolls back only the Deployment's Pod template]]: published events are one
  more thing outside the Pod template that a rollback leaves as the new version left it.
