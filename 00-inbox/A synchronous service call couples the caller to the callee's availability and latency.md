---
tags: [microservices, communication, messaging, interview]
status: draft
author: claude
source: "https://learn.microsoft.com/en-us/azure/architecture/microservices/design/interservice-communication"
created: 2026-09-30
score: 0.88
review: "borderline"
score_reasons: ["atomic: 0.57 (borderline)"]
judged_by: "jev-1.13.0"
judge_rounds: 2
---
# A synchronous service call couples the caller to the callee's availability and latency

## Core idea
In synchronous communication a service calls an API that another service exposes, over a
protocol such as HTTP or gRPC, and waits for the response. The operation therefore fails when
the downstream service is unavailable, and in a chain where A calls B and B calls C, each wait
adds to the latency the original caller sees. Blocked calls also hold threads and connections
in the caller, so a slow callee can cascade the failure upstream. Asynchronous messaging is the
contrast that removes this coupling: the sender does not wait, and a consumer that was down
picks up the messages when it recovers.

## Why choose / why not
- Choose a synchronous call when: the caller needs the answer to continue, such as a price or a
  permission check, and failing the request is acceptable while the callee is down. Give it a
  timeout and a circuit breaker so a slow callee cannot pin the caller's threads.
- Choose asynchronous messaging when: the caller only announces that something happened, the
  work can finish later, and the consumer being down must not fail the caller's request.
- Don't choose messaging when: the user must see the result in the same request; request-reply
  over a broker needs a reply queue and correlation IDs to do what one HTTP call does.

## Interview angle
- Probed as "service B is down: what happens to service A?" Walk the failure: with a
  synchronous call A's request fails or waits; with a broker A's send succeeds and B catches up
  when it recovers.
- Common wrong answer: "we use WebClient, so the call is asynchronous." Asynchronous I/O only
  frees the calling thread; HTTP is still a synchronous protocol, so A still needs B up to
  finish.
- Strong answer: name the trade. Messaging buys failure isolation and pays with duplicates to
  handle, no result in the same call, and a broker to operate.

## Related
- [[Microservices and messaging MOC]]: this note opens the "Microservices and messaging" section,
  whose question is what happens when the other side is slow, down, or sees the message twice.
