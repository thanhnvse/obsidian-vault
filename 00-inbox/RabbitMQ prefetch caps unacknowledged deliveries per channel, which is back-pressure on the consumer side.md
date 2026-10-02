---
tags: [system-design, interview, messaging, rabbitmq, back-pressure]
status: draft
author: claude
up: ["[[Microservices and messaging MOC]]"]
source: "https://www.rabbitmq.com/docs/confirms"
created: 2026-10-01
score: 0.87
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# RabbitMQ prefetch caps unacknowledged deliveries per channel, which is back-pressure on the consumer side

## Core idea
A queue protects a consumer only if something limits how much work the broker pushes to it at once. In RabbitMQ that limit is the channel prefetch count, set with basic.qos: once a channel holds that many unacknowledged deliveries, RabbitMQ stops delivering more messages on that channel until at least one of the outstanding ones is acknowledged. Without a bound the broker pushes as fast as it can: a prefetch value of 0 means no limit, and in automatic acknowledgement mode consumers can be overwhelmed by the rate of deliveries, accumulating a backlog in memory and running out of heap. The RabbitMQ documentation says prefetch values in the 100 through 300 range usually offer optimal throughput without significant risk of overwhelming consumers.

## Why choose / why not
- Use manual acknowledgements with a bounded prefetch when: each message does real work, such as a database write; the backlog stays in the broker instead of the consumer's heap, and a crash requeues only the deliveries in flight.
- Lower the prefetch towards 1 when: messages are slow and uneven in cost and several consumers compete; a large prefetch lets one consumer hoard messages while others sit idle.
- Don't count on prefetch alone when: consumers autoscale; prefetch bounds each channel, so the total load on the database still grows with the number of consumers.

## Interview angle
- Probed as "consumers run out of memory during a spike; why?", or "what does back-pressure mean in a queue-based system?"
- Common wrong answer: "we use auto-ack for speed", which removes both the delivery limit and the redelivery after a crash.
- Strong answer: manual ack after the work commits, a bounded prefetch per channel, a cap on consumer count matched to what the database can take, and shedding at the producer when the backlog keeps growing.

## Related
- [[A queue in front of a service levels load spikes at the cost of an immediate response]]: that note caps how many consumers run; prefetch caps how much work each one holds, the other half of the same limit.
- [[Committing Kafka offsets after processing gives at-least-once delivery, so consumers must be idempotent]]: acknowledging after the work is what makes an unacknowledged delivery come back after a crash, the same ordering that gives Kafka consumers at-least-once delivery.
