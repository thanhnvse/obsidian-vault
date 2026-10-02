---
tags: [java, spring, bean-lifecycle, transactions, interview]
status: draft
author: claude
source: "https://docs.spring.io/spring-framework/reference/core/beans/factory-nature.html"
created: 2026-09-30
score: 0.871
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# @Transactional does not apply inside @PostConstruct because the proxy is created after initialisation

## Core idea
Spring calls a bean's init callbacks only after the bean has received all its dependencies, and
brackets them with post-processors: `postProcessBeforeInitialization`, then `@PostConstruct`,
`afterPropertiesSet()` and a custom init method, then `postProcessAfterInitialization`. Spring
AOP auto-proxying is itself a `BeanPostProcessor`, and post-processors that wrap a bean in a
proxy normally do it in that last step. The init callback therefore runs on the raw bean, before
any AOP interceptor is applied. A `@PostConstruct` method marked `@Transactional`, or one that
calls the same bean's `@Transactional` methods, runs without a transaction.

## Why choose / why not
- Keep work in `@PostConstruct` when: it validates configuration or builds in-memory structures
  from injected values, with no database access; the Spring reference limits init methods to
  exactly that.
- Move startup work that reads or writes the database out of it: run it after the context is
  ready, from `SmartInitializingSingleton.afterSingletonsInstantiated()`, a
  `ContextRefreshedEvent` listener or Spring Boot's `ApplicationReadyEvent`, and open the
  transaction through another bean's proxy or a `TransactionTemplate`.
- Don't put `@Transactional` on the `@PostConstruct` method itself: it is ignored without a
  warning, and the code seems to work until a rollback is needed.

## Interview angle
- Probed as "walk me through the bean lifecycle"; a senior answer says where the proxy appears,
  not only the names of the callbacks.
- Common wrong answer: "`@PostConstruct` runs on the finished bean, so its annotations work."
- Strong answer: init callbacks see the raw object, the auto-proxy creator wraps it in
  `postProcessAfterInitialization`, so startup work that needs a transaction runs after the
  context refresh.

## Related
- [[A checked exception commits a Spring @Transactional method by default]]: another case where
  a `@Transactional` method does not behave as the annotation suggests; there the proxy applies
  the rollback rules, here the proxy does not exist yet.
- [[@Transactional MOC]]: answers the Spring Boot cluster's question "what does the
  container do to my bean, and when?" for the moment the proxy is created.
