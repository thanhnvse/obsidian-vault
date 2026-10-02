---
tags: [java, spring, transactions, aop, interview]
status: draft
author: claude
source: "https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/annotations.html"
created: 2026-09-30
score: 0.88
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Self-invocation bypasses the Spring @Transactional proxy

## Core idea
In proxy mode, the default, Spring applies `@Transactional` through a proxy around the bean, and
only external calls that come in through that proxy are intercepted. A call from one method of
the bean to another, `this.method()`, runs on the target object, so the called method's
annotation has no effect and no transaction is started for it. When the calling method has no
transaction either, each Spring Data repository call such as `save()` runs in its own
transaction and commits, so a write followed by an exception stays committed. AspectJ mode
weaves the transactional behaviour into the class bytecode, so it also covers self-invocation.

## Why choose / why not
- Move the transactional method to another bean when: the two methods are really two use cases;
  the call then goes through a proxy, and the split usually exposes a missing boundary. This is
  the default fix.
- Use `TransactionTemplate` when: the boundary has to sit inside one method or around part of
  it; the transaction is explicit code, so there is no proxy to bypass.
- Don't inject the bean into itself (a `@Lazy` self-reference) unless a refactor is not possible
  now: it works, but it hides the design problem and surprises readers.
- Choose AspectJ mode only when: self-calls are everywhere and the team accepts compile-time or
  load-time weaving in the build.

## Interview angle
- Probed as "why didn't my transaction start?"; self-invocation is the cause to name right after
  a checked exception.
- Common wrong answer: "put `@Transactional` on the class"; class-level attributes still work
  through the proxy, so internal calls are still not intercepted.
- Strong answer: say the proxy is the transaction, then derive the siblings from it: `final`
  methods, work handed to another thread, and objects created with `new`.

## Related
- [[A checked exception commits a Spring @Transactional method by default]]: the other first
  answer to "why didn't my transaction roll back?"; there the proxy runs and applies its rollback
  rules, here the call never reaches the proxy.
- [[@Transactional does not apply inside @PostConstruct because the proxy is created after initialisation]]:
  the same root cause at a different moment; during initialisation the proxy does not exist yet,
  and with self-invocation it exists but the call goes around it.
