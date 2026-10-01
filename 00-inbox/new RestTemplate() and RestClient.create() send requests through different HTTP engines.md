---
tags: [java, spring, http-client, interview]
status: draft
author: claude
up: ["[[Spring MOC]]"]
source: "https://docs.spring.io/spring-framework/docs/6.1.x/javadoc-api/org/springframework/web/client/RestClient.Builder.html"
created: 2026-10-01
score: 0.897
review: "ready"
score_reasons: ["possible conflict with draft [[RestClient replaces RestTemplate as Spring's synchronous HTTP client]] (p=0.71)", "numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# new RestTemplate() and RestClient.create() send requests through different HTTP engines

## Core idea
`RestTemplate` inherits its request factory from `HttpAccessor`, whose default is
`SimpleClientHttpRequestFactory`, built on the JDK's `HttpURLConnection`. A `RestClient` built
without a request factory picks one from the classpath instead: Apache HttpClient or Jetty's client
if present, otherwise the JDK `HttpClient` from the `java.net.http` module. On a classpath with
neither Apache nor Jetty, `new RestTemplate()` and `RestClient.create()` therefore run on two
different JDK clients, each with its own proxy, TLS, connection-reuse and timeout settings.
`RestClient.create(restTemplate)` instead reuses the template's request factory, interceptors and
message converters. A lab on Spring Framework 6.1.2 confirmed both defaults: `HttpURLConnection` for
`new RestTemplate()` and the JDK `HttpClient` for `RestClient.create()`.

## Why choose / why not
- Set the request factory explicitly, the same for both clients, when: a service uses both during a
  migration, or relies on proxy, TLS, pooling or timeout settings; otherwise behaviour depends on
  the client type and on what is on the classpath.
- Migrate with `RestClient.create(restTemplate)` when: the existing template's engine and timeouts
  are already configured; the new client keeps them.
- Don't assume that a `RestTemplate` and a `RestClient` built with defaults behave the same because
  both are synchronous Spring clients.

## Interview angle
- Probed as "we replaced `RestTemplate` with `RestClient.create()` and our proxy settings stopped
  applying; why?".
- Common wrong answer: "`RestClient` is a fluent wrapper over the same engine."
- Strong answer: the defaults differ, `HttpURLConnection` against the JDK `HttpClient` when neither
  Apache nor Jetty is present; set the request factory explicitly, or build the client from the
  existing template.

## Related
- [[RestClient replaces RestTemplate as Spring's synchronous HTTP client]]: partial overlap; that
  note says both clients share the same request factory types and how to wrap a template, while this
  one shows that their default engines differ.
- [[RestClient and WebClient take their timeouts from the underlying HTTP library]]: because the
  timeouts belong to the engine, two default engines mean two different places to configure them.
