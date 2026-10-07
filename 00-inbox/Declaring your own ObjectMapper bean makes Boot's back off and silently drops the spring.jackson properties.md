---
tags: [java, spring, spring-boot, auto-configuration, interview]
status: draft
author: claude
up: ["[[Spring MOC]]"]
source: ""
created: 2026-10-07
review: unjudged
---
# Declaring your own ObjectMapper bean makes Boot's back off and silently drops the spring.jackson properties

## Core idea
Boot's auto-configured beans back off when you declare a bean of the same type, because
`@ConditionalOnMissingBean` matches by type, not by name. The `spring.*` properties that configured
Boot's bean stop applying, because they belonged to it. In a lab on Spring Boot 3.2.1,
`spring.jackson.default-property-inclusion=non_null` made Boot's `ObjectMapper` omit nulls; after
the test declared its own `ObjectMapper`, the same property was silently (âm thầm) ignored and nulls
were written again. A `Jackson2ObjectMapperBuilderCustomizer` bean instead keeps Boot's mapper, with
its properties and your change. Any bean of type `Executor` likewise removes
`applicationTaskExecutor`, and `spring.task.execution.pool.core-size` then applies to nothing.

## Why choose / why not
- Set a property when: the authors expose one (`spring.jackson.*`, `spring.task.execution.*`); it is
  one line and every other default stays.
- Add a customizer bean when: a property cannot express the change and a customizer exists for the
  bean (only the common beans have one); it extends Boot's bean instead of replacing it.
- Declare your own bean only when: you want exactly your construction and nothing of Boot's;
  configure the replacement fully, because the `spring.*` settings no longer reach it.
- Use `spring.autoconfigure.exclude` when: a whole auto-configuration is wrong for the application;
  everything it provided is gone, including beans that others need.

## Interview angle
- Asked as "I added my own `ObjectMapper` and my `spring.jackson` settings stopped working. Why?";
  the same shape appears when `@Async` runs on a different pool after someone declares an
  `Executor`.
- Common wrong answer: treating the new bean as extra configuration on top of Boot's, so the
  properties should still apply.
- Strong answer: Boot's bean backed off and the properties belonged to it; the ladder is property,
  customizer, own bean, exclude; the proof is the conditions report (`--debug`) or the Actuator
  `conditions` endpoint, which names the bean that made it back off, instead of "I think it backs
  off".

## Related
- [[Spring MOC]]: the map for "what does the container do to my bean"; this note covers the case
  where a bean Boot would create is replaced by yours.
- [[@ConditionalOnMissingBean sees your beans only in auto-configuration, because Boot defers that import until your configuration is parsed]]:
  why your bean wins in the first place; this note is about what winning costs.
- [[In Spring Boot an environment variable or command-line argument beats the config file, key by key]]:
  the other half of overriding Boot: where a property value comes from when several sources set it.

Written up in win-interview: backend/java/docs/spring-boot-fundamentals.md, sections 2.3 and 3.1
