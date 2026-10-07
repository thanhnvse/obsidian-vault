---
tags: [java, spring, spring-mvc, aop, interview]
status: draft
author: claude
up: ["[[Spring MOC]]"]
source: "https://github.com/spring-projects/spring-framework/blob/v6.1.2/framework-docs/modules/ROOT/pages/web/webmvc/mvc-servlet/handlermapping-interceptor.adoc"
created: 2026-10-07
review: unjudged
---
# A servlet filter sees every request, an interceptor only requests that found a handler, and an aspect only method calls on a bean

## Core idea
All three wrap the controller, at different depths. A servlet `Filter` sees the raw
`HttpServletRequest` and `HttpServletResponse` of every request mapped to it and can pass wrappers
down the chain. A `HandlerInterceptor` runs inside the `DispatcherServlet` once a handler is chosen,
so it sees the handler (usually a `HandlerMethod` with its annotations) but never a request the
`HandlerMapping` rejects, such as a `consumes` mismatch. An aspect sees a method call on a proxied
bean and knows nothing about HTTP; a request rejected in argument resolution, such as malformed JSON
or a failed `@Valid`, never reaches it, so 400s and 415s are missing from its counts.

## Why choose / why not
- Use a filter when: the concern is about HTTP traffic as a whole, including 404s and requests that
  later fail: correlation id and MDC, access logs, compression, authentication (Spring Security is a
  filter chain). It does not know which handler will run.
- Use an interceptor when: the check needs the handler or an annotation on the controller method,
  such as per-handler timing. It is not a security layer, and `postHandle` cannot change a JSON
  response because the response is already committed; use a `ResponseBodyAdvice` or a filter.
- Use an aspect when: the concern is about what the application does and should cover service
  methods too: metrics, auditing, retries, transactions. Put it on the service layer; it is blind to
  rejected requests and has the proxy limits.
- Don't cast `handler` to `HandlerMethod` in an interceptor without a check: unknown URLs reach it
  with a `ResourceHttpRequestHandler`, and the 404 becomes a 500.

## Interview angle
- Probed as "filter, interceptor or aspect: when do you use which?", and inside "walk me through a
  request to a Spring Boot REST endpoint".
- Common wrong answer: "interceptors and filters are the same thing."
- Strong answer: say what each can see (raw servlet objects for every request; the handler, inside
  the `DispatcherServlet`; a bean method call with no HTTP), give one example and one limit each,
  then say who can see which failure: an aspect never sees a 400, and `postHandle` never changes a
  committed response.

## Related
- [[Spring AOP applies @Aspect annotations through proxies, without the AspectJ weaver]]: the aspect
  column rests on its proxy limits: only method calls on Spring beans, and calls through `this` are
  not advised.
- [[Self-invocation bypasses the Spring @Transactional proxy]]: the proxy limit behind "no
  self-invocation" in the aspect's why-not.
- [[An exception thrown in a servlet filter never reaches an @ExceptionHandler, so the client gets Spring Boot's own error body]]:
  what follows from a filter sitting outside the `DispatcherServlet`: its exceptions reach no
  handler.
- [[Spring MOC]]: the nearest broader map; no web-layer map exists, so this note joins it directly
  through its own `up:` link.

Written up in win-interview: backend/java/docs/spring-mvc.md, sections 2.1 to 2.4 and 3
