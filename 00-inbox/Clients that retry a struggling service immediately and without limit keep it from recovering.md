---
tags: [microservices, resilience, retry, communication, interview]
status: draft
author: claude
up: ["[[Microservices and messaging MOC]]"]
source: "https://learn.microsoft.com/en-us/azure/architecture/antipatterns/retry-storm/"
created: 2026-10-01
score: 0.903
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Clients that retry a struggling service immediately and without limit keep it from recovering

## Core idea
When a service becomes unavailable or busy, every client that retries its failed requests adds load
to the service that is trying to recover. The Azure Architecture Center calls this the Retry Storm
antipattern: retries within a short period of time are unlikely to succeed because the service
likely hasn't recovered, and excessive connection attempts during recovery can overwhelm the
service and intensify the original problem, a situation sometimes called a thundering herd.
Retrying forever is also pointless, because requests typically remain valid only for a limited
time. The page's client-side fixes are to limit the number of retry attempts and their duration,
to pause between attempts and increase the wait, for example with exponential backoff, to use the
Circuit Breaker pattern, and to wait for the period a `Retry-After` response header gives. A
service protects itself by throttling requests at a gateway and sending `Retry-After` to clients.
An error that marks the request itself as invalid, such as `400 Bad Request`, is not worth
retrying at all.

## Why choose / why not
- Retry with a small attempt limit and exponential backoff when: the failure is transient, such as
  a timeout or a `503`, the operation is idempotent, and the caller can still answer within its own
  deadline.
- Don't retry when: the error says the request is invalid, such as a `400`, or the operation is not
  idempotent and the first attempt may already have succeeded, such as a timed-out `POST` without
  an idempotency key.
- Add a circuit breaker when: many clients call the same dependency, so that during an outage they
  stop calling it at full rate instead of each one retrying on its own schedule.

## Interview angle
- Probed as "the payment service had a five-minute outage, recovered, and fell over again right
  away; why?" Every client retried at once as it came back.
- Common wrong answer: "just retry until it works." Immediate, unlimited retries multiply the load
  on the service that is trying to recover, and they also repeat non-idempotent work.
- Strong answer: bounded attempts, exponential backoff with jitter so clients do not retry in step,
  a circuit breaker, honouring `Retry-After`, and retries only for idempotent operations.

## Related
- [[A synchronous service call couples the caller to the callee's availability and latency]]: that
  note names timeouts and circuit breakers as the limits on synchronous coupling; this note is what
  goes wrong when the retry part of that toolkit has no bounds.
- [[A cache stampede happens when a hot key expires and many requests regenerate it at once]]: the
  same shape of many callers hitting one resource at the same moment, at a cache miss instead of
  at a service that is recovering.
