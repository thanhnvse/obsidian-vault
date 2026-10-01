---
tags: [java, spring, bean-lifecycle, interview]
status: draft
author: claude
source: "https://docs.spring.io/spring-framework/reference/core/beans/factory-scopes.html"
created: 2026-09-30
score: 0.92
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Spring does not call @PreDestroy on prototype-scoped beans

## Core idea
For a prototype-scoped bean, the container creates a new instance every time the bean is
requested, by injection or by `getBean()`, hands it to the client and keeps no further record of
it. Initialisation callbacks such as `@PostConstruct` run for beans of every scope, but
destruction callbacks, such as `@PreDestroy`, `DisposableBean.destroy()` or a custom destroy
method, are not called for prototypes. The client code must clean up prototype-scoped objects
and release the resources they hold. The Spring reference describes the container's role for a
prototype as a replacement for the Java `new` operator.

## Why choose / why not
- Choose prototype scope for stateful, short-lived objects that hold no resource needing
  release, such as a per-use builder or accumulator; the reference recommends prototype for
  stateful beans and singleton for stateless ones.
- Don't make a bean that holds a connection, a thread pool or a file handle a prototype and
  expect `@PreDestroy` to release it; keep it a singleton, whose `@PreDestroy` runs when the
  context closes, or make it `AutoCloseable` and close it with try-with-resources in the caller.
- If the container must still release prototype resources, register a custom `BeanPostProcessor`
  that keeps references to the beans needing cleanup, as the reference suggests; it is more code,
  so prefer the options above.

## Interview angle
- Probed as "does `@PreDestroy` always run on shutdown?" or "what is the lifecycle of a
  prototype bean?".
- Common wrong answer: "yes, Spring calls `@PreDestroy` on every bean when the context closes."
- Strong answer: the container manages a prototype up to the hand-over, like `new`; after that
  the caller owns it, including its cleanup.

## Related
- [[@Transactional does not apply inside @PostConstruct because the proxy is created after initialisation]]:
  the other end of the same lifecycle; the init callbacks and post-processors described there
  still run for prototypes, and only the destruction phase is skipped.
