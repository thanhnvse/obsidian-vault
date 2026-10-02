---
tags: [java, spring, dependency-injection, interview]
status: draft
author: claude
source: "https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/autowired.html"
created: 2026-09-30
score: 0.877
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Injecting a List of an interface gives every bean of that type, sorted by @Order

## Core idea
When an injection point is a `List<T>` or an array of `T`, Spring injects every bean in the
context that matches type `T`. The elements are sorted by the `Ordered` interface, `@Order` or
`@Priority` on the target beans, and otherwise follow the registration order of the bean
definitions. `@Order` affects only that order at injection points; it does not change singleton
startup order, which follows dependencies and `@DependsOn`. With no matching bean, a required
collection injection point fails, except in a bean with a single constructor, where the
collection argument resolves to an empty list.

## Why choose / why not
- Choose it when every implementation must run in a defined order, such as validators, enrichers
  or filters in a pipeline: a new step is a new bean, and the consumer does not change.
- Declare `@Order` explicitly whenever the order matters: registration order depends on
  component scanning and configuration, so it is not a contract.
- Don't use it when exactly one implementation is picked per call; inject a `Map<String, T>`
  keyed by bean name, or use a qualifier. Don't introduce the interface for a list of one.

## Interview angle
- Probed as "how would you add a validation rule without touching the service?".
- Common wrong answer: "`@Order` controls the order in which beans are created."
- Strong answer: collection injection plus an explicit `@Order`, then the single-constructor
  empty-list rule, and that `@Order` has nothing to do with startup order.

## Related
- [[Spring MOC]]: part of the Spring Boot cluster's question "what does the
  container do to my bean, and when?"; here the container resolves one dependency to many beans.
