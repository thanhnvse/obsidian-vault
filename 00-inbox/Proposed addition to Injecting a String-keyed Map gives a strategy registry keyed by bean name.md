---
tags: [java, spring, dependency-injection, interview]
status: draft
author: claude
up: ["[[Spring MOC]]"]
proposes_for: "[[Injecting a String-keyed Map gives a strategy registry keyed by bean name]]"
source: "https://docs.spring.io/spring-framework/docs/6.1.x/javadoc-api/org/springframework/context/annotation/ConfigurationClassPostProcessor.html"
created: 2026-09-30
score: 0.808
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 2
---
# Proposed addition to Injecting a String-keyed Map gives a strategy registry keyed by bean name

## Core idea
The default bean name, and therefore the map key, also depends on how the class is registered.
`ConfigurationClassPostProcessor` uses a bean name generator with fully qualified class names as
default bean names for configuration-level imports, while component scanning uses the plain
`AnnotationBeanNameGenerator`. An unnamed class picked up by scanning is keyed by its uncapitalised
simple name, such as `bankTransferPayment`. The same class registered through `@Import` is keyed
by its fully qualified class name, such as `com.example.payment.BankTransferPayment`.

## Why choose / why not
- Choose explicit names (`@Component("bank-transfer")`) or keys declared by the strategy itself
  whenever a class could be imported as well as scanned, for example in a library's
  auto-configuration or in a test configuration.
- Don't assume a test that `@Import`s a strategy sees the same keys as the scanned application.

## Interview angle
- Probed as a follow-up: "the registry works in the app but returns null in the test. Why?".
- Strong answer: the test imports the class, so its bean name is the fully qualified class name.

## Related
- [[Injecting a String-keyed Map gives a strategy registry keyed by bean name]]: the note this
  addition is proposed for. Its rule for default names, the uncapitalised simple class name, holds
  only for scanned beans; this addition gives the `@Import` case, where the same class gets a
  different key.
- [[Injecting a List of an interface gives every bean of that type, sorted by @Order]]: the way out
  of the problem; a registry built from the injected list, keyed by a method each strategy declares,
  does not depend on bean names at all, so scanning versus importing stops mattering.
