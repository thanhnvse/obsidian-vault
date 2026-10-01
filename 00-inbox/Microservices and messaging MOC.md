---
tags: [moc, microservices, messaging, interview]
type: moc
status: draft
author: claude
up: ["[[Java backend interview MOC]]"]
created: 2026-09-30
---
# Microservices and messaging MOC

The question behind this map: *what happens when the other side is slow, down, or sees the message twice?*

## Service calls
- [[A synchronous service call couples the caller to the callee's availability and latency]]: sync vs async in one sentence

## Kafka
- [[Kafka orders records only within a partition, and the record key chooses the partition]]: ordering, and why the record key decides it
- [[The partition count caps how many consumers in a Kafka consumer group can do work]]: parallelism, and its ceiling
- [[Committing Kafka offsets after processing gives at-least-once delivery, so consumers must be idempotent]]: delivery semantics
- [[Kafka exactly-once covers Kafka-to-Kafka processing, not external side effects]]: the limit of exactly-once
