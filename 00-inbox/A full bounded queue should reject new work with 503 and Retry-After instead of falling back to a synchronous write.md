---
tags: [system-design, load-shedding, backpressure, http, interview]
status: draft
author: claude
up: ["[[System design MOC]]"]
source: "https://www.rfc-editor.org/rfc/rfc9110.html#name-503-service-unavailable"
created: 2026-10-08
review: "unjudged"
score_reasons: ["jev unavailable: HTTP 401 after 1 attempt(s): {\"detail\":{\"error_type\":\"authentication_error\",\"message\":\"Cannot authenticate with the server. Please check your API key and try again.\"}}"]
judge_rounds: 1
---
# A full bounded queue should reject new work with 503 and Retry-After instead of falling back to a synchronous write

## Core idea
A bounded queue in front of slow work, such as an object-store write or a database insert, protects
the service only if a full queue changes what callers see. Falling back to doing the work
synchronously when the queue is full moves the slow path onto request threads exactly when the
system is overloaded, so latency climbs for every request instead of only the excess ones.
Rejecting at the edge, with `503 Service Unavailable` and a `Retry-After` header, keeps latency for
accepted requests bounded and moves the wait to the clients, which retry later. It is safe only
when retries are idempotent, so the same event sent twice is stored once.

## Why choose / why not
- Shed load with 503 and `Retry-After` when: clients can retry (SDKs, trackers, background
  senders) and every write carries an idempotency key.
- Use 429 instead when: the limit is a per-client quota rather than a system-wide overload.
- Don't shed when: the caller cannot retry and the work must not be lost; then the queue needs more
  capacity or a durable spill, not rejection.

## Interview angle
- Asked as "your ingestion queue is full; what should the API do?".
- Common wrong answer: "write straight to the database so nothing is lost".
- Strong answer: reject fast with 503 and `Retry-After`, cap the request body size before parsing
  it, and make writes idempotent so client retries are safe.

## Related
- [[A queue in front of a service levels load spikes at the cost of an immediate response]]: that
  note is what the queue buys; this one is what to do when the queue itself is full.
- [[A ThreadPoolExecutor starts threads beyond its core size only when the queue refuses the task, so an unbounded queue makes maximumPoolSize irrelevant]]:
  the in-process version; a bounded queue with a rejection policy is what makes overload visible.
- [[A retried POST is safe only when the server claims its Idempotency-Key atomically before the work and replays the stored result]]:
  load shedding hands retries to clients, so the write path must be idempotent.
- [[System design MOC]]: the map entry for behaviour under overload.
- Seen in: LEO-CDP/leo-customer360, customer360-event-api/core/buffered_storage.py and
  core/routers/tracking.py, which answer 503 with `Retry-After` when the tracking stream is full
  (read 2026-10-08).
