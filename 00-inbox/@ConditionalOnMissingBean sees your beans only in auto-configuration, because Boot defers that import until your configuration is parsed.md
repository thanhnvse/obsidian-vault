---
tags: [java, spring, spring-boot, auto-configuration, interview]
status: draft
author: claude
up: ["[[Spring MOC]]"]
source: "https://docs.spring.io/spring-boot/docs/3.2.1/reference/html/features.html#features.developing-auto-configuration.condition-annotations.bean-conditions"
created: 2026-10-07
review: unjudged
---
# @ConditionalOnMissingBean sees your beans only in auto-configuration, because Boot defers that import until your configuration is parsed

## Core idea
`@EnableAutoConfiguration` imports the classes named in each jar's
`META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` through a
deferred selector, so Spring parses every other configuration class first. A condition sees
only what is registered so far, and the deferral puts your beans there before
`@ConditionalOnMissingBean` asks, so your bean wins and Boot's default backs off. In a lab on Boot
3.2.1 the back-off still worked when the user configuration was registered after the one that
enables auto-configuration, but the same class imported with a plain `@Import` left both beans in
the context, because its condition ran before the user bean was registered.

## Why choose / why not
- Use `@ConditionalOnMissingBean` when: you write an auto-configuration, a library default meant to
  be replaced; its import is deferred, so the condition sees every user bean.
- Don't use it in your own `@Configuration`: the result depends on the order the classes happen to
  be processed, so one machine can end with one bean and another with two; use `@Primary` or
  `@Qualifier` instead.
- Add `@AutoConfiguration(after = Other.class)` when: one auto-configuration needs the beans of
  another; without it a `@ConditionalOnBean` on the class that sorts first does not see the later
  one's bean.

## Interview angle
- Probed as "how does auto-configuration work, and how do I override it?"; the mechanism answer is
  "deferred import, so a condition can see your bean".
- Common wrong answers: "auto-configuration scans the classpath for `@Configuration` classes at
  runtime", and "`@ConditionalOnMissingBean` works anywhere".
- Strong answer: an imports file in each jar, a deferred import after your configuration, a
  condition per bean, back-off for your bean; and that the same annotation in your own configuration
  depends on processing order.

## Related
- [[Spring MOC]]: the parent map; this note gives the mechanism behind the question "which bean
  exists, and why".
- [[Declaring your own ObjectMapper bean makes Boot's back off and silently drops the spring.jackson properties]]:
  the practical cost of the back-off: the properties that configured Boot's bean stop applying.

Written up in win-interview: backend/java/docs/spring-boot-fundamentals.md, section 2.2
