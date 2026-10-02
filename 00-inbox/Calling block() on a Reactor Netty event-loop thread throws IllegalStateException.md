---
tags: [java, spring, webflux, http-client, interview]
status: draft
author: claude
up: ["[[Spring MOC]]"]
source: "https://projectreactor.io/docs/core/release/api/reactor/core/scheduler/NonBlocking.html"
created: 2026-10-01
score: 0.891
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Calling block() on a Reactor Netty event-loop thread throws IllegalStateException

## Core idea
Reactor marks its non-blocking threads, including the Reactor Netty event-loop threads that run
`WebClient` response callbacks, with the `NonBlocking` marker interface. Reactor's blocking APIs,
such as `Mono.block()`, `Flux.blockFirst()` and `Flux.blockLast()`, detect that marker and throw an
`IllegalStateException` instead of parking the thread. A `block()` inside a `map` on a `WebClient`
response therefore fails, with a message saying that blocking is not supported in a
`reactor-http-nio` thread. On a servlet request thread the same `block()` succeeds and holds that
thread until the response arrives. A lab on Spring Framework 6.1.2 with Reactor Netty 1.1.14 and
Reactor Core 3.6.1 reproduced the failure.

## Why choose / why not
- Compose dependent calls with `flatMap`, `zipWith` or `Mono.zip` when: a reactive pipeline needs
  the result of another call; the event loop is then never parked.
- Move unavoidable blocking work, such as a JDBC call, onto `Schedulers.boundedElastic()` with
  `publishOn` or `subscribeOn` when: a reactive pipeline has to call a blocking library.
- Use `RestClient` instead when: every `WebClient` call would end in `block()` on a servlet thread;
  the code is synchronous anyway.

## Interview angle
- Probed as "what happens if you call `block()` inside a `WebClient` pipeline?".
- Common wrong answer: "it waits for the result, like a synchronous call."
- Strong answer: on Reactor's own threads it throws, because one parked event loop would stall
  every connection it serves; compose instead, or move blocking work to a bounded elastic
  scheduler.

## Related
- [[WebClient saves threads only when the caller does not block on it]]: partial overlap; that note
  covers blocking on the caller's thread, which wastes the advantage, and this one covers blocking
  on Reactor's own thread, which fails.
