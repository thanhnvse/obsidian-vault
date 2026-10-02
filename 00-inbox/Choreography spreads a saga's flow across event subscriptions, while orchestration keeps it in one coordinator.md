---
tags: [microservices, saga, consistency, messaging, interview]
status: draft
author: claude
up: ["[[Microservices and messaging MOC]]"]
source: "https://learn.microsoft.com/en-us/azure/architecture/patterns/saga"
created: 2026-09-30
score: 0.847
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Choreography spreads a saga's flow across event subscriptions, while orchestration keeps it in one coordinator

## Core idea
A saga runs a business transaction as a sequence of local transactions in different services,
and the open question is where the sequence itself lives. The Azure Architecture Center describes
two answers. In choreography, services exchange events without a centralized controller: each
local transaction publishes domain events that trigger local transactions in other services, so
the flow exists only as the sum of the subscriptions. In orchestration, a centralized orchestrator
tells each participant which operation to perform, stores the state of each task, and handles
failure recovery with compensating transactions, so the flow lives in one component. The same
page lists the trade-off: choreography needs no extra service and has no single point of failure
but becomes confusing as steps are added, risks cyclic dependencies and needs every service
running for integration tests, while orchestration suits complex workflows at the cost of
coordination logic and an orchestrator that is itself a point of failure.

## Why choose / why not
- Choose choreography when: the flow has few services and steps and needs no coordination logic,
  such as two services reacting independently to an `OrderPlaced` event.
- Choose orchestration when: the flow has many steps or compensations, or keeps changing, because
  the order of the steps and the failure handling then live in one place.
- Don't keep choreography once nobody can say which service reacts to which event: move the flow
  into an orchestrator before the next step is added.

## Interview angle
- Probed as "choreography or orchestration for checkout?" Decide by the number of steps and
  compensations, and name the orchestrator's availability as the cost.
- Common wrong answer: "choreography is more decoupled, so it is always better." The flow becomes
  invisible as it grows, cycles creep in, and testing needs every service.
- Strong answer: walk through the same failure in both styles: who notices that the inventory
  reservation failed, and who triggers the refund.

## Related
- [[A synchronous service call couples the caller to the callee's availability and latency]]:
  that note explains why services move from synchronous calls to messaging; a saga is how a
  business transaction stays consistent once its steps are asynchronous.
- [[Committing Kafka offsets after processing gives at-least-once delivery, so consumers must be idempotent]]:
  saga steps that are triggered by messages get redelivered like any consumer, so every step and
  every compensation must be idempotent.
- [[Microservices and messaging MOC]]: the map this note sits under, in its service-calls section.
