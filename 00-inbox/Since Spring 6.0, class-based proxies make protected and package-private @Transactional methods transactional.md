---
tags: [java, spring, transactions, aop, proxies, interview]
status: draft
author: claude
up: ["[[@Transactional MOC]]"]
source: "https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/annotations.html"
created: 2026-10-01
score: 0.88
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Since Spring 6.0, class-based proxies make protected and package-private @Transactional methods transactional

## Core idea
As of Spring Framework 6.0, `@Transactional` on a protected or package-private method takes effect
by default when the bean has a class-based proxy, because the CGLIB subclass can override that
method and run the transaction around it. Up to Spring Framework 5.3, with the standard
configuration, the same annotation was ignored without an error, and only public methods were
transactional. The 6.0 rule covers class-based proxies only: on an interface-based JDK proxy, a
transactional method must still be public and declared on the proxied interface. A lab on Spring
Framework 6.1.2 with Spring Boot 3.2.1, which uses class-based proxies by default, confirmed that a
package-private `@Transactional` method called through the injected bean ran in a transaction.

## Why choose / why not
- Rely on it when: the project is on Spring Framework 6 or later with class-based proxies, the
  Spring Boot 3 default, and a transactional method should stay package-private so that only its
  package can call it.
- Keep transactional methods public when: the code may run on Spring 5, or under JDK proxies
  (`spring.aop.proxy-target-class=false`); there a non-public annotation is silently ignored.
- Don't make a method private to "protect" its transaction: a private method is never advised, so
  it never gets one.

## Interview angle
- Probed as "does `@Transactional` work on a non-public method?".
- Common wrong answer: "only public methods are transactional"; that was the rule up to 5.3 and is
  out of date for class-based proxies.
- Strong answer: name the version and the proxy type: since 6.0 CGLIB proxies advise protected and
  package-private methods, JDK proxies only public interface methods, private methods never; and
  only calls that come in through the proxy count.

## Related
- [[Spring Boot proxies a bean with CGLIB even when the bean implements an interface]]: that
  default is why the 6.0 visibility rule applies to most Spring Boot beans.
- [[Self-invocation bypasses the Spring @Transactional proxy]]: visibility decides whether the
  proxy can advise a method, self-invocation whether the call reaches the proxy at all; a
  package-private method called through `this` still has no transaction.
