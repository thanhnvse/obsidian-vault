---
tags: [java, spring, aop, proxies, interview]
status: draft
author: claude
up: ["[[Spring MOC]]"]
source: "https://github.com/spring-projects/spring-framework/blob/v6.1.2/spring-aop/src/main/java/org/springframework/aop/framework/CglibAopProxy.java"
created: 2026-09-30
score: 0.867
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A final method on a CGLIB-proxied Spring bean runs on the proxy instance, where injected fields are null

## Core idea
A CGLIB proxy is a generated subclass of the bean's class that overrides its methods, runs the
advice and then calls the separate target object. A `final` method cannot be overridden, so a call
to it executes the original method body on the proxy instance itself instead of being routed to
the target. Spring creates the CGLIB proxy instance through Objenesis, so the class's constructor
does not run for it and the proxy's own fields stay uninitialised. The final method therefore runs
with no advice, such as no transaction, and sees `null` in the fields that hold its dependencies.
Spring reports this only at DEBUG level, warning that such calls "will NOT be routed to the target
instance and might lead to NPEs against uninitialized fields in the proxy instance".

## Why choose / why not
- Choose non-final methods when the bean carries advice (`@Transactional`, `@Async`, `@Cacheable`,
  an aspect): it is the only fix that keeps Spring Boot's CGLIB default, and it costs nothing. In
  Kotlin, where functions are final by default, the `kotlin-spring` compiler plugin does it for you.
- Choose an interface plus JDK proxies (`spring.aop.proxy-target-class=false`) when the method or
  class must stay final, for example a type you do not own: a JDK proxy overrides nothing and
  delegates every interface call to the target. The cost is that injecting the class fails.
- Choose AspectJ weaving only when final methods must be advised in place: it changes the bytecode
  instead of subclassing, at the price of a compiler plugin or an agent in every build.
- Don't choose `final` on a Spring service "so nobody overrides it": the container is the one
  subclass that has to.

## Interview angle
- Probed as "a repository field is `null` in one method of my service, but injected everywhere
  else. Why?".
- Common wrong answer: "the injection failed", or "make the method final so nobody overrides it".
- Strong answer: there are two objects, the proxy and the target; Objenesis skips the constructor,
  and a final method cannot be routed to the target, so it runs on the empty proxy instance.

## Related
- [[Self-invocation bypasses the Spring @Transactional proxy]]: the other way a call misses the
  advice; there the call never reaches the proxy, here it reaches the proxy instance but is not
  routed to the target.
- [[Spring Boot proxies a bean with CGLIB even when the bean implements an interface]]: why the
  CGLIB rules apply to most Boot beans, interface or not.
