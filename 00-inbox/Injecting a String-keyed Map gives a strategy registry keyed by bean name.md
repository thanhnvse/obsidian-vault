---
tags: [java, spring, dependency-injection, design-patterns, interview]
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
# Injecting a String-keyed Map gives a strategy registry keyed by bean name

## Core idea
Spring can autowire a `Map<String, T>`: the values are all beans of type `T` and the keys are
their bean names. A consumer can then pick an implementation at runtime with `get(key)` instead
of an `if`/`else` chain or a `switch` over types, and a new strategy is a new bean with no change
to the consumer. For a scanned component the bean name is the one given in the stereotype
annotation, such as `@Service("card")`, and otherwise the uncapitalised simple class name, so an
unnamed `CardPayment` class is registered as `cardPayment`.

## Why choose / why not
- Choose it when the implementation is selected at runtime by a key the caller already has, such
  as a payment method code, and the set of implementations keeps growing.
- Give each bean an explicit name, or build the map yourself from an injected `List<T>` keyed by
  a method such as `type()`: with default names, renaming a class silently changes the key and
  the lookup returns `null`.
- Don't use it for two fixed variants: a plain `if` is clearer, and a lookup by string moves a
  compile-time choice to runtime.
- Handle an unknown key explicitly: `get` returns `null`, so fail loudly and name the known keys.

## Interview angle
- Probed as "how would you remove this `switch` over payment types?".
- Common wrong answer: a factory class with the same `switch` moved inside it.
- Strong answer: a `Map<String, T>` injection or a map built from `List<T>`, then how an unknown
  key fails, and why explicit keys beat names derived from the class.

## Related
- [[Injecting a List of an interface gives every bean of that type, sorted by @Order]]: the same
  collection-injection mechanism; a list runs every implementation in order, a map picks one by
  key, and the list is the input when you build the map with your own keys.
