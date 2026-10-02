---
tags: [java, spring, aop, proxies, interview]
status: draft
author: claude
up: ["[[Spring MOC]]"]
source: "https://docs.spring.io/spring-framework/reference/core/aop/ataspectj.html"
created: 2026-10-01
score: 0.794
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Spring AOP applies @Aspect annotations through proxies, without the AspectJ weaver

## Core idea
Spring's @AspectJ support interprets the same annotations as AspectJ 5, such as `@Aspect`,
`@Pointcut` and `@Around`, and uses a library supplied by AspectJ for pointcut parsing and
matching. The advice itself runs through Spring AOP's runtime proxies, and there is no dependency
on the AspectJ compiler or weaver. The `aspectjweaver` jar must still be on the classpath, but only
for those annotations and the pointcut matching, so its presence does not mean that any bytecode is
woven. An `@Aspect` applied this way inherits the proxy limits: Spring AOP supports only method
execution join points on Spring beans, and calls through `this` are not advised. A lab on Spring
Framework 6.1.2 with AspectJ weaver 1.9.21 ran `@Aspect` advice with no weaving configured.

## Why choose / why not
- Choose Spring AOP with `@Aspect` when: the concern applies to method calls on Spring beans, such
  as metrics, auditing or tracing around services; it needs no build change.
- Choose AspectJ compile-time or load-time weaving when: the advice must also cover
  self-invocation, objects created with `new`, constructors or field access; it needs the AspectJ
  compiler or a weaving agent in every build and run.
- Don't move to AspectJ weaving just to fix one self-invoked method: moving the method to another
  bean puts the call back through a proxy.

## Interview angle
- Probed as "does Spring AOP use AspectJ?" or "is AspectJ required for `@Aspect`?".
- Common wrong answer: "`@Aspect` means the classes are woven by AspectJ."
- Strong answer: the annotations and the pointcut language come from AspectJ, the runtime is Spring
  proxies; that is why an aspect sees only bean method calls that go through the proxy, and why
  weaving is a separate, opt-in mode.

## Related
- [[Self-invocation bypasses the Spring @Transactional proxy]]: the same proxy limit applies to a
  custom `@Aspect`, and that note names AspectJ mode as the way around it.
- [[Spring Boot proxies a bean with CGLIB even when the bean implements an interface]]: the kind of
  proxy that carries an `@Aspect`'s advice in a Spring Boot application.
