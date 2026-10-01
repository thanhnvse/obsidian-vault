---
tags: [system-design, interview, messaging, queue, reliability]
status: draft
author: claude
up: ["[[Microservices and messaging MOC]]"]
source: "https://learn.microsoft.com/en-us/azure/service-bus-messaging/service-bus-dead-letter-queues"
created: 2026-10-01
score: 0.856
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A message that fails on every delivery needs a delivery limit and a dead-letter queue, or it is retried forever

## Core idea
A poison message fails every time it is processed, for example because it is malformed or needs a resource that is not available. Under at-least-once delivery a failed message goes back to the queue, so without a limit it is redelivered again and again, burning consumer capacity on work that can never succeed. Microsoft's Competing Consumers pattern says the system should prevent such messages from returning to the queue indefinitely and should store their details elsewhere for analysis. Brokers do this with a delivery limit and a dead-letter queue. Azure Service Bus moves a message to the dead-letter queue with the reason MaxDeliveryCountExceeded once its delivery count exceeds the maximum, 10 by default. Amazon SQS moves a message to the dead-letter queue named in the redrive policy after it has been received maxReceiveCount times. RabbitMQ quorum queues, since RabbitMQ 4.0, have a default delivery limit of 20, after which a message is dropped, or dead-lettered if a dead-letter exchange is configured.

## Why choose / why not
- Set the delivery limit high enough for transient failures when: errors such as timeouts usually pass on a retry; SQS warns that a maxReceiveCount of 1 moves a message after a single failure.
- Dead-letter at once from the application when: the failure is permanent, such as a malformed payload; Service Bus calls this application-level dead-lettering and recommends the exception type as the reason and the stack trace as the description.
- Don't put a dead-letter queue behind a FIFO queue when: the exact order of messages must hold; moving one message aside breaks the sequence, as the SQS guide warns.
- Give the dead-letter queue an alert and an owner when: you enable one at all; Service Bus never cleans it up automatically, so an unread dead-letter queue is silent data loss.

## Interview angle
- Probed as "how do you handle a message that always fails?"
- Common wrong answer: "failed messages are retried until they succeed"; a poison message never succeeds, so it loops.
- Strong answer: retry transient failures with backoff up to a delivery limit, dead-letter permanent ones at once with the reason recorded, alert on the dead-letter queue, and redrive after the fix.

## Related
- [[Committing Kafka offsets after processing gives at-least-once delivery, so consumers must be idempotent]]: redelivery is what at-least-once delivery promises; this note is the case where redelivery can never succeed, so something must stop it.
- [[A queue in front of a service levels load spikes at the cost of an immediate response]]: the queue in that note is where a poison message loops when nothing limits its deliveries, eating the consumer capacity the queue was sized for.
