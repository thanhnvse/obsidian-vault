---
tags: [java, spring, dependency-injection, interview]
status: draft
author: claude
up: ["[[Spring MOC]]"]
source: "https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/autowired.html"
created: 2026-10-01
score: 0.907
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# An injected Map of beans keeps registration order and ignores @Order

## Core idea
When Spring resolves a `Map<String, T>` injection point, it collects every bean of type `T`, keyed
by bean name, in the order in which their bean definitions were registered, and returns that map
without sorting it. `@Order`, `Ordered` and `@Priority` on those beans therefore do not change the
map's iteration order; the Spring reference describes that sorting only for items in an array or a
list. An injected array or `List<T>` of the same beans is sorted by the container's order
comparator. Code that iterates such a map and expects the `@Order` sequence runs the beans in
registration order instead, which can change when a bean moves to another configuration class or
package. A lab on Spring Framework 6.1.2 confirmed that an injected map kept registration order
although its beans carried different `@Order` values.

## Why choose / why not
- Inject a `Map<String, T>` when: the consumer picks one bean by key and never depends on the
  iteration order.
- Inject a `List<T>` when: the beans must run in a sequence, such as validation steps; only lists
  and arrays are sorted.
- Build a `LinkedHashMap` from an injected `List<T>` when: you need both a key lookup and a defined
  order; it keeps the list's sorted order.

## Interview angle
- Probed as "is an injected `Map` ordered by `@Order`?", usually after a question on `List`
  injection.
- Common wrong answer: "yes, the same as a `List`."
- Strong answer: lists and arrays are sorted by the order comparator, maps are keyed by bean name in
  registration order and never sorted, so anything ordered gets a `List`.

## Related
- [[Injecting a String-keyed Map gives a strategy registry keyed by bean name]]: partial overlap;
  that note covers what the keys are, this one covers the iteration order, which `@Order` does not
  affect.
- [[Injecting a List of an interface gives every bean of that type, sorted by @Order]]: the list
  injection that does apply the order, and the input for a map with a defined order.
