---
tags: [java, spring, transactions, concurrency, interview]
status: draft
author: claude
up: ["[[@Transactional MOC]]"]
source: "https://docs.spring.io/spring-framework/docs/6.1.x/javadoc-api/org/springframework/transaction/annotation/Transactional.html"
created: 2026-10-01
score: 0.873
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Work handed to another thread runs outside the caller's Spring transaction

## Core idea
`@Transactional` is commonly used with thread-bound transactions managed by a
`PlatformTransactionManager`: the transaction and its resources, such as the JDBC connection or
the JPA `EntityManager`, are bound to the current thread, and every data access call on that
thread joins them. The `@Transactional` javadoc states that this transaction does not propagate
to threads newly started within the method. Work handed to another thread, such as an `@Async`
method or a `CompletableFuture.supplyAsync` task, therefore runs with no active transaction from
the caller and uses a connection of its own. Its writes are not part of the caller's transaction,
so the caller's rollback does not undo them, and under READ COMMITTED or a stricter level it
cannot see the caller's uncommitted writes. A lab on Spring Framework 6.1.2 confirmed that
`TransactionSynchronizationManager.isActualTransactionActive()` was `true` on the calling thread
and `false` inside a `CompletableFuture.supplyAsync` task started from the same method.

## Why choose / why not
- Keep the work on the calling thread when: it must commit or roll back together with the
  caller's writes; Spring's thread-bound transaction cannot span threads.
- Start the background work after the commit, for example from an `AFTER_COMMIT`
  `@TransactionalEventListener` or through an outbox, when: it must read the caller's data;
  started inside the transaction it can run before the commit and miss the new rows.
- Give the background work its own `@Transactional` method on another bean when: it writes
  several rows that must be atomic among themselves; it then gets its own transaction and
  connection.

## Interview angle
- Probed as "I call an `@Async` method from my `@Transactional` service. Is it in the same
  transaction?".
- Common wrong answer: "yes, it is called from inside the transaction."
- Strong answer: the transaction lives in thread-bound state, so the async task runs without it,
  on another connection, unable to see uncommitted rows; then say how to start it only after the
  commit.

## Related
- [[Self-invocation bypasses the Spring @Transactional proxy]]: another way code inside a
  transactional method runs without the transaction; there the call skips the proxy, here the
  proxy ran but its transaction stays on the calling thread.
- [[@TransactionalEventListener moves a side effect after the commit but loses it if the process dies]]:
  the usual way to start background work only once the transaction has committed, with its own
  failure mode.
- [[REQUIRES_NEW keeps the outer connection while it borrows a second one from the pool]]: the
  same pool cost appears here, because the other thread borrows a second connection while the
  caller still holds its own.
