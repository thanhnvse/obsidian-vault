---
tags: [java, spring, http-client, resilience, interview]
status: draft
author: claude
up: ["[[Spring MOC]]"]
source: "https://projectreactor.io/docs/netty/1.1.14/api/reactor/netty/http/client/HttpClient.html"
created: 2026-09-30
score: 0.727
review: "parked"
score_reasons: ["why_choose: 0.39 (fail)"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# An HTTP client's read timeout is measured between reads, so it does not bound the whole call

## Core idea
`URLConnection.setReadTimeout` raises a `SocketTimeoutException` when the timeout expires before
there is data available for read, so the wait is measured from one read to the next. Reactor
Netty's `responseTimeout` is defined the same way: the maximum duration allowed between each
network-level read operation while reading a given response. A server that sends a slow trickle of
bytes, each within the timeout, never trips either setting, so a call can last far longer than the
configured value. Bounding the whole exchange takes a separate deadline on the call, such as
Reactor's `Mono.timeout` operator for a `WebClient` call.

## Why choose / why not
- Set both when the caller has a latency budget: the read or response timeout detects a peer that
  has gone silent, and the overall deadline enforces the budget.
- Size the overall deadline from the caller's own deadline, so the downstream call gives up before
  the caller's caller does.
- Don't present the read timeout as the maximum call duration in a design or an SLA discussion.

## Interview angle
- Probed as "your read timeout is two seconds, so why did this call take thirty?".
- Common wrong answer: "the timeout did not apply"; it did, but it restarts with every read.
- Strong answer: name what each timeout measures (connect, between reads, whole call) and which
  failure each one bounds.

## Related
- [[RestClient and WebClient take their timeouts from the underlying HTTP library]]: partial
  overlap; that note covers where the timeouts are configured and that the defaults are infinite,
  this one adds what the read or response timeout measures and why an overall deadline is still
  needed on top of it.
