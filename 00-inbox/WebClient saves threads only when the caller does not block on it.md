---
tags: [java, spring, webflux, http-client, microservices, interview]
status: draft
author: claude
source: "https://docs.spring.io/spring-framework/reference/web/webflux-webclient.html"
created: 2026-09-30
score: 0.877
review: "borderline"
score_reasons: ["atomic: 0.58 (borderline)", "numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 2
---
# WebClient saves threads only when the caller does not block on it

## Core idea
`WebClient` is the non-blocking, reactive HTTP client of Spring WebFlux, built on Reactor and
available since Spring Framework 5.0. Its advantage is high concurrency with fewer hardware
resources, because no thread waits while a request is in flight. Calling `block()` on its
`Mono` or `Flux` gives that advantage back: the calling thread waits for the response, just as
it would with a synchronous client. The Spring reference therefore says that with `Flux` or
`Mono` you should never have to block in a Spring MVC or WebFlux controller; return the
reactive type instead.

## Why choose / why not
- Choose `WebClient` when: the service runs on WebFlux, or one request fans out to several
  remote calls that can run concurrently and be combined before the response, so no thread
  waits on each call in turn.
- Don't choose it when: the service is Spring MVC, makes one remote call per request, and would
  call `block()` right away; `RestClient` gives the same result without the Reactor
  programming model.

## Interview angle
- Probed as "is `WebClient` faster than `RestTemplate`?" It is not faster per call; it serves
  more concurrent calls with fewer threads, and only while nothing blocks.
- Common wrong answer: "we switched to `WebClient` and added `.block()`, so we are reactive
  now." Each request still holds a thread for the whole remote call.
- Strong answer: name where the thread is saved (no thread parked on I/O) and where it is lost
  (`block()` on the request thread), then pick the client by stack.

## Related
- [[RestClient replaces RestTemplate as Spring's synchronous HTTP client]]: `RestClient` is
  where this note sends blocking Spring MVC code, so the two notes together answer "which
  Spring HTTP client?".
- [[A synchronous service call couples the caller to the callee's availability and latency]]:
  non-blocking I/O saves threads but does not remove that coupling, because HTTP stays a
  synchronous protocol.
