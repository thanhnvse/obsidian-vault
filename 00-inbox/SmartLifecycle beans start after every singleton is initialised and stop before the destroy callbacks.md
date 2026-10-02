---
tags: [java, spring, bean-lifecycle, interview]
status: draft
author: claude
up: ["[[Spring MOC]]"]
source: "https://docs.spring.io/spring-framework/reference/core/beans/factory-nature.html"
created: 2026-09-30
score: 0.853
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# SmartLifecycle beans start after every singleton is initialised and stop before the destroy callbacks

## Core idea
When the application context is refreshed, after all objects have been instantiated and
initialised, the default lifecycle processor starts every `SmartLifecycle` bean whose
`isAutoStartup()` returns `true`. Objects with the lowest `getPhase()` value start first, and on
shutdown the reverse order is followed. On a regular shutdown, all `Lifecycle` beans receive a stop
notification before the general destruction callbacks, such as `@PreDestroy`, are propagated. The
Spring reference warns that this is not guaranteed on a hot refresh or a stopped refresh attempt,
where only destroy methods are called.

## Why choose / why not
- Choose it for components that run: message consumers, schedulers, pollers. They start once
  everything they use is wired, and stop consuming before those resources are destroyed.
- Use phases to order dependent components: a consumer in a higher phase than the component it
  feeds starts after it and stops before it.
- Don't start such work in `@PostConstruct`: that runs per bean while the context is still
  creating later beans, and nothing stops it in order at shutdown.
- Don't count on `stop()` after a failed startup: make the destroy callback safe on its own.

## Interview angle
- Probed as "where would you start a Kafka or JMS consumer in a Spring application?".
- Common wrong answer: "in `@PostConstruct`".
- Strong answer: `SmartLifecycle` with a phase, started after all singletons and stopped before
  destruction on a regular shutdown, plus the refresh-failure caveat.

## Related
- [[@Transactional does not apply inside @PostConstruct because the proxy is created after initialisation]]:
  `@PostConstruct` runs on the raw bean before its proxy exists, while `SmartLifecycle.start()` runs
  after every bean, proxies included, is ready.
- [[Spring does not call @PreDestroy on prototype-scoped beans]]: the destruction phase that the
  stop notification precedes, and its scope limit.
