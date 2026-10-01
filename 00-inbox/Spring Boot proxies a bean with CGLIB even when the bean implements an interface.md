---
tags: [java, spring, aop, proxies, interview]
status: draft
author: claude
up: ["[[Spring MOC]]"]
source: "https://docs.spring.io/spring-boot/reference/features/aop.html"
created: 2026-09-30
score: 0.889
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Spring Boot proxies a bean with CGLIB even when the bean implements an interface

## Core idea
Plain Spring AOP uses a JDK dynamic proxy when the target object implements at least one
interface, and a CGLIB subclass otherwise. Spring Boot's auto-configuration changes that default:
it configures Spring AOP to use CGLIB proxies, and setting `spring.aop.proxy-target-class` to
`false` switches back to JDK proxies. A JDK proxy implements only the bean's interfaces, so it is
not an instance of the bean's class. With JDK proxies, an injection point typed with the concrete
class therefore fails at startup, while a CGLIB proxy is a subclass and satisfies both the
interface and the class.

## Why choose / why not
- Keep the Boot default when any code or test injects beans by their class: the proxy type then
  does not change when someone adds an interface to a class.
- Choose JDK proxies only when every injection point uses an interface, or when an advised class
  is `final` but implements an interface: CGLIB cannot subclass a final class, a JDK proxy does not
  need to.
- Don't switch the property to "enforce" programming to interfaces in a codebase that injects
  classes: the failure appears only at startup, as `BeanNotOfRequiredTypeException`.

## Interview angle
- Probed as "JDK proxy or CGLIB: which one does Spring use?".
- Common wrong answer: "JDK proxies whenever there is an interface"; that is plain Spring's rule,
  not Spring Boot's default.
- Strong answer: the decision rule, Boot's override and the reason for it (the bean stays an
  instance of its class), then what breaks when the property is set to `false`.

## Related
- [[Self-invocation bypasses the Spring @Transactional proxy]]: holds for both proxy types; the
  proxy type decides what can be injected, not whether calls from inside the bean are intercepted.
- [[A final method on a CGLIB-proxied Spring bean runs on the proxy instance, where injected fields are null]]:
  the cost of the CGLIB default; a subclass cannot override final methods.
