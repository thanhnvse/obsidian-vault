---
tags: [java, spring, transactions, interview]
status: draft
author: claude
source: "https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/tx-propagation.html"
created: 2026-09-30
score: 0.853
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Catching an exception from a joined REQUIRED method ends in UnexpectedRollbackException

## Core idea
With `PROPAGATION_REQUIRED`, the default, an inner `@Transactional` method called through a
proxy while a transaction is active gets its own logical transaction scope but shares the
caller's physical transaction. When the inner method throws an exception that matches its
rollback rules, Spring marks the shared transaction rollback-only rather than rolling it back
there. If the outer method catches the exception and returns normally, its commit finds the
marker, rolls back all the work, the outer writes included, and throws
`UnexpectedRollbackException`. Spring does this on purpose, so that the caller is never misled
into assuming that a commit happened.

## Why choose / why not
- Let the inner exception propagate when: the inner failure means the use case failed; the whole
  transaction rolls back and the caller sees the real exception instead of
  `UnexpectedRollbackException`.
- Give the inner call `REQUIRES_NEW` when: the outer work must really continue after the inner
  failure, such as a best-effort audit record; the inner work then rolls back on its own, at the
  cost of a second connection.
- Return a result instead of throwing when: the inner "failure" is an expected business outcome
  that the caller handles; nothing is marked rollback-only, so the outer transaction commits.

## Interview angle
- Probed as "what is `UnexpectedRollbackException`?" or "I caught the exception, so why did
  everything roll back?".
- Common wrong answer: "catching the exception keeps the transaction alive."
- Strong answer: separate logical from physical transactions; each `REQUIRED` method has its own
  scope and rollback-only status, all scopes map to one physical transaction, and only the outer
  scope commits it.

## Related
- [[A checked exception commits a Spring @Transactional method by default]]: its rollback rules
  decide whether the inner exception marks the shared transaction at all; a checked exception
  thrown by the inner method does not.
- [[Self-invocation bypasses the Spring @Transactional proxy]]: the inner method gets its own
  scope only when the call goes through a proxy; called through `this`, there is no inner
  interceptor to set the marker.
