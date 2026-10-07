---
tags: [java, spring, spring-mvc, error-handling, interview]
status: draft
author: claude
up: ["[[Spring MOC]]"]
source: "https://docs.spring.io/spring-boot/docs/3.2.x/reference/html/web.html#web.servlet.spring-mvc.error-handling"
created: 2026-10-07
review: unjudged
---
# An exception thrown in a servlet filter never reaches an @ExceptionHandler, so the client gets Spring Boot's own error body

## Core idea
Exception handlers live inside the `DispatcherServlet`; filters run before it. An exception thrown
by a filter (Spring Security's filter chain included), a call to `response.sendError` and an
exception that escapes the servlet all end in the container, which makes an `ERROR` dispatch to
Spring Boot's `/error`. The client then gets Boot's `BasicErrorController` JSON, not your
`application/problem+json`: two error formats. A `@ResponseStatus` exception class that no advice
method matches ends the same way, because `ResponseStatusExceptionResolver` answers with `sendError`. To get one format, replace `/error`
with your own `ErrorController` bean, which makes Boot's back off, or add an `ErrorAttributes` bean.

## Why choose / why not
- Replace `/error` (your own `ErrorController` or an `ErrorAttributes` bean) when: the API promises
  one error format; no advice can reach a filter's exception, a `sendError` or an escaped exception.
- Map exceptions in the advice, not with `@ResponseStatus` on the class, when: you want a
  `ProblemDetail`; the annotation ends in `sendError`, so the client gets Boot's body even with
  `spring.mvc.problemdetails.enabled=true`.
- Register your own `AuthenticationEntryPoint` and `AccessDeniedHandler` through
  `exceptionHandling(...)` when: Spring Security's filter-level 401 and 403 must use the same
  format; they come from its own handlers, not from your advice (sourced from the Spring Security
  6.2.1 source, not run in the lab).
- Don't expect detail in the default body: it has four members (`timestamp`, `status`, `error`,
  `path`), and `server.error.include-message` and `server.error.include-binding-errors` default to
  `never` in 3.2.1, so neither the message nor the field errors appear.

## Interview angle
- Probed as "does `@ControllerAdvice` catch every exception?", and as "why did my
  `@ExceptionHandler` not run?", where a filter's exception is the second cause to rule out.
- Common wrong answer: "`@ControllerAdvice` catches every exception in the application."
- Strong answer: only exceptions thrown inside the `DispatcherServlet`; name what escapes (filter
  exceptions, `sendError`, `@ResponseStatus` classes, anything unmapped), close the second path with
  a replaced `/error`, and add that a `@PreAuthorize` denial is thrown inside the
  `DispatcherServlet`, so the advice sees it first.

## Related
- [[The first @ControllerAdvice with any matching handler wins, even over a more specific handler in a later advice]]:
  the other common reason a handler does not run: another advice with a match is ordered first.
- [[A servlet filter sees every request, an interceptor only requests that found a handler, and an aspect only method calls on a bean]]:
  why a filter sits outside the `DispatcherServlet` in the first place.
- [[Spring MOC]]: the nearest broader map; no web-layer map exists, so this note joins it directly
  through its own `up:` link.

Written up in win-interview: backend/java/docs/spring-mvc.md, sections 4.1 and 4.4
