---
tags: [java, spring, dependency-injection, interview]
status: draft
author: claude
up: ["[[Spring MOC]]"]
source: "https://docs.spring.io/spring-framework/docs/6.1.x/javadoc-api/org/springframework/beans/factory/ObjectProvider.html"
created: 2026-10-01
score: 0.892
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Only ObjectProvider.orderedStream() sorts beans by @Order, while stream() does not

## Core idea
`ObjectProvider<T>` offers two streams over all beans of type `T`. Its javadoc defines
`orderedStream()` as pre-ordered according to the factory's common order comparator, which is the
same `Ordered`, `@Order` and `@Priority` sort that an injected `List<T>` gets. `stream()` comes
without specific ordering guarantees, typically in registration order. Code that replaces an
injected `List<T>` with `ObjectProvider<T>.stream()`, for example to resolve the beans lazily,
therefore loses the `@Order` sequence without any error. A lab on Spring Framework 6.1.2 confirmed
that `orderedStream()` matched the order of the injected list while `stream()` kept registration
order.

## Why choose / why not
- Use `orderedStream()` when: the beans run as a sequence, such as filters or checks, and the
  provider replaces a `List<T>` injection.
- Use `stream()` when: order does not matter, such as collecting the beans into a map by a key they
  declare; it skips the sort.
- Prefer an injected `List<T>` when: the beans are always needed; the dependency stays visible in
  the constructor, and a missing or broken bean fails the startup instead of the first call.

## Interview angle
- Probed as "how do you get every bean of a type lazily, in `@Order`?".
- Common wrong answer: "`ObjectProvider.stream()` returns them sorted, like a `List`."
- Strong answer: `orderedStream()` applies the order comparator and `stream()` does not; then weigh
  lazy resolution against a dependency that the constructor no longer shows.

## Related
- [[Injecting a List of an interface gives every bean of that type, sorted by @Order]]: the eager
  injection whose order `orderedStream()` reproduces and `stream()` does not.
