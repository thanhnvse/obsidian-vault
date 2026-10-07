---
tags: [java, spring, spring-mvc, error-handling, interview]
status: draft
author: claude
up: ["[[Spring MOC]]"]
source: "https://github.com/spring-projects/spring-framework/blob/v6.1.2/framework-docs/modules/ROOT/pages/web/webmvc/mvc-controller/ann-exceptionhandler.adoc"
created: 2026-10-07
review: unjudged
---
# The first @ControllerAdvice with any matching handler wins, even over a more specific handler in a later advice

## Core idea
For an exception thrown inside the `DispatcherServlet`, `ExceptionHandlerExceptionResolver` tries
the controller's own `@ExceptionHandler` methods first, then each advice bean in order (`@Order`);
no order means lowest precedence. The first advice with any matching method wins, even if a later
advice has a more specific one; only inside one class does the closest exception type win. A
catch-all `@ExceptionHandler(Exception.class)` in an advice ordered first therefore shadows
every other mapping: in the lab (Spring 6.1.2) a 404 became a 500 until the specific advice
came first. Boot's `ProblemDetailsExceptionHandler` (with `spring.mvc.problemdetails.enabled=true`)
sits at order 0, so an unordered advice of yours never handles `MethodArgumentNotValidException`.

## Why choose / why not
- Put the catch-all in the same advice class as the specific mappings when: you want a generic 500
  as a safety net; inside one class the closest type wins, so it cannot shadow the others, including
  the ones `ResponseEntityExceptionHandler` declares.
- Don't put a catch-all in a separate advice ordered first: every error, including Spring's own 400s
  and 415s, becomes a 500, and validation failures show up in the 5xx metrics.
- Extend `ResponseEntityExceptionHandler` in your advice when: `spring.mvc.problemdetails.enabled`
  is on; Boot's handler then backs off. Otherwise order your advice before 0.
- With a catch-all, map `@ResponseStatus` exception classes explicitly and rethrow
  `AccessDeniedException` and `AuthenticationException`; otherwise the catch-all turns their
  annotated status, or a `@PreAuthorize` 403, into a 500.

## Interview angle
- Probed as "why did my `@ExceptionHandler` not run?"; another advice with any match ordered
  earlier, such as a catch-all or Boot's handler at order 0, is the most likely cause.
- Common wrong answer: "the most specific `@ExceptionHandler` always wins."
- Strong answer: local handlers first, then advice beans in order with the first match winning, and
  the closest type only within one class; then how you would notice it (validation failures in the
  5xx metrics) and the fix (the catch-all in the main advice, advices ordered on purpose).

## Related
- [[An exception thrown in a servlet filter never reaches an @ExceptionHandler, so the client gets Spring Boot's own error body]]:
  the next cause of "my handler did not run": the exception never entered the `DispatcherServlet`,
  so no advice could see it.
- [[@ConditionalOnMissingBean sees your beans only in auto-configuration, because Boot defers that import until your configuration is parsed]]:
  the same back-off mechanism: Boot's handler is registered only when no
  `ResponseEntityExceptionHandler` bean exists, which is why extending that class makes it step
  aside.
- [[Spring MOC]]: the nearest broader map; no web-layer map exists, so this note joins it directly
  through its own `up:` link.

Written up in win-interview: backend/java/docs/spring-mvc.md, sections 4.2 and 4.4
