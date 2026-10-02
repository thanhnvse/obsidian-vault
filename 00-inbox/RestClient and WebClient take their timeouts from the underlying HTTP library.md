---
tags: [java, spring, http-client, resilience, microservices, interview]
status: draft
author: claude
source: "https://docs.spring.io/spring-framework/reference/web/webflux-webclient/client-builder.html"
created: 2026-09-30
score: 0.808
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# RestClient and WebClient take their timeouts from the underlying HTTP library

## Core idea
`RestClient` sends requests through a `ClientHttpRequestFactory` that adapts an HTTP library
such as the JDK `HttpClient`, Apache HttpComponents, Jetty or Reactor Netty, and `WebClient`
does the same through a `ClientHttpConnector`. Timeouts are therefore configured on that
library: `JdkClientHttpRequestFactory#setReadTimeout` sets the JDK request timeout, and for
`WebClient` on Reactor Netty the Spring reference sets `ChannelOption.CONNECT_TIMEOUT_MILLIS`
and `HttpClient#responseTimeout`. The library defaults do not bound the wait: the Java 21
Javadoc of `HttpRequest.Builder#timeout` says that not setting a timeout is the same as an
infinite one, and the Reactor Netty 1.3 reference says `responseTimeout` is not specified by
default.

## Why choose / why not
- Set both timeouts on every client bean when: the client calls another service. The connect
  timeout bounds a host that does not accept the connection; the response timeout bounds a
  host that accepted it but does not answer.
- Put them in the one shared client, not at each call site: a `RestClient` is thread-safe once
  built, so one configured bean per callee means no call site can forget the timeout.
- Don't rely on `Mono#timeout` or a block timeout alone: the Spring and Reactor Netty
  references both recommend the library's own timeout settings, which act at a lower level and
  can bound the connect and response phases separately.

## Interview angle
- Probed as "what is the default timeout of your HTTP client?" Name the library behind the
  client, state its default, then say the service does not rely on it.
- Common wrong answer: "Spring sets a sensible default." Spring's request factories pass the
  setting through, and the JDK request timeout is infinite when unset.
- Strong answer: a connect timeout and a response timeout per client, sized from the callee's
  observed latency, and one place in the code where they are set.

## Related
- [[A synchronous service call couples the caller to the callee's availability and latency]]:
  the response timeout is what bounds that latency coupling; without one, the callee's worst
  latency becomes the caller's.
- [[WebClient saves threads only when the caller does not block on it]]: on Reactor Netty the
  `responseTimeout` is what bounds a `WebClient` call, whether or not the caller blocks on it.
