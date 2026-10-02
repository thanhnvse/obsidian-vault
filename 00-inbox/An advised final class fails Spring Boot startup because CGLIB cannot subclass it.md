---
tags: [java, spring, aop, proxies, kotlin, interview]
status: draft
author: claude
up: ["[[Spring MOC]]"]
source: "https://docs.spring.io/spring-framework/reference/core/aop/proxying.html"
created: 2026-10-01
score: 0.873
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# An advised final class fails Spring Boot startup because CGLIB cannot subclass it

## Core idea
A CGLIB proxy is a runtime-generated subclass of the bean's class, so a class declared `final`
cannot get one. When advice such as `@Transactional`, `@Cacheable` or an aspect matches a final
class under class-based proxying, which Spring Boot uses by default, the context fails at startup
with "Could not generate CGLIB subclass of class". Under that default, an interface on the final
class does not help, because Spring still tries to subclass it. Java records are always final and
Kotlin classes are final by default, so both hit this unless Kotlin's `kotlin-spring` compiler
plugin opens the annotated classes. With `spring.aop.proxy-target-class=false`, a final class that
implements an interface gets a JDK dynamic proxy instead and starts. A lab on Spring Framework 6.1.2
with Spring Boot 3.2.1 confirmed both outcomes.

## Why choose / why not
- Remove `final` from a class that carries advice when: the project keeps Spring Boot's CGLIB
  default; it costs nothing and keeps injection by class working.
- Apply the `kotlin-spring` plugin in Kotlin projects whose beans carry Spring annotations: it
  opens classes annotated with `@Component`, `@Transactional`, `@Async` or `@Cacheable`, so they
  can be subclassed.
- Move the advice to a separate service when: the type is a record or must stay final; keep the
  record as plain data and annotate the service that uses it.
- Switch to JDK proxies only when: every injection point uses the interface; otherwise
  injection by class fails at startup instead.

## Interview angle
- Probed as "can Spring proxy a final class?" or "why does my Kotlin `@Transactional` service fail
  at startup?".
- Common wrong answer: "Spring falls back to a JDK proxy when the class implements an interface";
  under Spring Boot's default it does not.
- Strong answer: CGLIB needs a subclass, so a final advised class fails loudly at startup; records
  and Kotlin classes are the usual cases; a JDK proxy works only with an interface and the
  property set to `false`.

## Related
- [[A final method on a CGLIB-proxied Spring bean runs on the proxy instance, where injected fields are null]]:
  the same subclassing limit at method level, where the failure is silent instead of loud.
- [[Spring Boot proxies a bean with CGLIB even when the bean implements an interface]]: the
  default that explains why an interface does not rescue a final class in a Spring Boot
  application.
