---
tags: [java, spring, bean-lifecycle, interview]
status: draft
author: claude
up: ["[[Spring MOC]]"]
source: "https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-dependson.html"
created: 2026-10-01
score: 0.871
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Spring destroys a singleton before the beans it depends on

## Core idea
When the context closes, Spring destroys singletons in the reverse of their recorded dependency
order. Each time the container autowires bean B into bean A, or A declares `depends-on` B, it
registers A as a dependent of B, to be destroyed before B. A bean's destruction callbacks, such as
`@PreDestroy`, therefore run while the beans it depends on are still intact. The Spring reference documents the same rule for `depends-on`, for singleton beans only:
dependent beans are destroyed first, so `depends-on` also controls shutdown order. A dependency the
container never injected, such as a bean looked up with `getBean()` at run time, is not recorded,
so it can be destroyed before the bean that still uses it. A lab on Spring Framework 6.1.2
confirmed that a bean's `@PreDestroy` ran before that of the bean injected into it.

## Why choose / why not
- Inject the dependency when: a bean's `@PreDestroy` still needs it, such as a buffer flushed to a
  repository at shutdown; the injection is what makes the container destroy the user first.
- Declare `@DependsOn` when: one bean must be shut down before another but does not hold a
  reference to it; for singletons it orders both startup and shutdown.
- Don't use a bean found with `getBean()` in shutdown code: the container has no record of that
  dependency and may already have destroyed it.

## Interview angle
- Probed as "bean A uses bean B in its `@PreDestroy`. Is B still usable at that point?".
- Common wrong answer: "shutdown order is random" or "beans are destroyed in creation order."
- Strong answer: reverse dependency order, built from the dependencies the container recorded at
  injection or through `depends-on`, with the unrecorded `getBean()` lookup as the gap.

## Related
- [[SmartLifecycle beans start after every singleton is initialised and stop before the destroy callbacks]]:
  `stop()` runs before the destruction phase whose order this note describes.
- [[Spring does not call @PreDestroy on prototype-scoped beans]]: the order applies only to beans
  the container destroys, and prototypes are not among them.
