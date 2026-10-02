---
tags: [java, spring, http-client, microservices, interview]
status: draft
author: claude
source: "https://docs.spring.io/spring-framework/reference/integration/rest-clients.html"
created: 2026-09-30
score: 0.861
review: "borderline"
score_reasons: ["atomic: 0.57 (borderline)", "numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 2
---
# RestClient replaces RestTemplate as Spring's synchronous HTTP client

## Core idea
`RestClient` is a synchronous HTTP client with a fluent API, introduced in Spring Framework
6.1 as the modern replacement for `RestTemplate`'s template-method API. The Spring Framework
7.0 reference documentation states that `RestTemplate` is deprecated in favor of `RestClient`
and will be removed in a future version. The Spring team's published plan puts the formal
`@Deprecated` annotation in Spring Framework 7.1 and the removal in 8.0. Both clients share the
same request factories, interceptors and message converters, so the replacement changes the
call-site API rather than the HTTP plumbing.

## Why choose / why not
- Choose `RestClient` when: writing new blocking code on the servlet stack, or touching a
  `RestTemplate` call site anyway; it is the synchronous client the Spring team maintains.
- Migrate gradually when: a service has many `RestTemplate` call sites. Wrap the existing
  template with `RestClient.create(restTemplate)` to keep its request factory and interceptors,
  then move call sites one at a time.
- Don't choose `RestClient` when: the service runs on WebFlux or needs streaming; use
  `WebClient` there.

## Interview angle
- Probed as "which HTTP client would you use in a new Spring service, and why not
  `RestTemplate`?" Give the version facts: `RestClient` since 6.1, `RestTemplate` deprecated in
  the 7.0 docs, formal `@Deprecated` planned for 7.1.
- Common wrong answer: "`RestTemplate` is deprecated, so use `WebClient` everywhere." That pulls
  Reactor into a blocking application; the synchronous successor is `RestClient`.
- Strong answer: pick by stack, `RestClient` for Spring MVC and `WebClient` for WebFlux, and
  add that neither choice removes the need for explicit timeouts.

## Related
- [[A synchronous service call couples the caller to the callee's availability and latency]]:
  `RestClient` is the tool for the synchronous side of that trade, so every coupling named
  there applies to each call made with it.
- [[Spring MOC]]: the entry point for these interview topics; this note belongs
  to its "Microservices and messaging" section.
