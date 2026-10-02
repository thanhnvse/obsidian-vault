---
tags: [kafka, messaging, microservices, interview]
status: draft
author: claude
up: ["[[Microservices and messaging MOC]]"]
source: "https://kafka.apache.org/36/configuration/consumer-configs/"
created: 2026-10-01
score: 0.884
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# The default Kafka rebalance revokes every partition, while cooperative rebalancing revokes only the ones that move

## Core idea
A consumer group rebalances when a member joins or leaves. In Kafka 3.6 the default partition.assignment.strategy is [RangeAssignor, CooperativeStickyAssignor], which uses the RangeAssignor and therefore the eager protocol. Under the eager protocol every member's ConsumerRebalanceListener revocation callback runs at the start of the rebalance for all of its partitions, so the group stops processing until the new assignment arrives. The CooperativeStickyAssignor uses incremental cooperative rebalancing (KIP-429): members keep their partitions and revoke only those that must move, which a follow-up rebalance then hands over.

## Why choose / why not
- Choose the CooperativeStickyAssignor when: rebalances are frequent (rolling deploys, autoscaling) or partitions carry expensive state; most of the group keeps working through a rebalance.
- Don't switch casually when: not every member supports it yet; the upgrade needs a rolling bounce that removes the RangeAssignor, and each membership change then costs two rebalances.

## Interview angle
- Probed as "why does the whole consumer group stall on every deploy?"
- Common wrong answer: "rebalances are always stop-the-world."
- Strong answer: eager default revokes everything; cooperative revokes only what moves; name the rolling-bounce migration.

## Related
- [[A Kafka consumer slower than max.poll.interval.ms loses its partitions to another member, and its offset commit fails]]: one common trigger of the rebalances this note is about.
- [[The partition count caps how many consumers in a Kafka consumer group can do work]]: the assignment a rebalance recomputes.
