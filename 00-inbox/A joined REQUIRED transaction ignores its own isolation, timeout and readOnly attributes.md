---
tags: [java, spring, transactions, interview]
status: draft
author: claude
source: "https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/tx-propagation.html"
created: 2026-09-30
score: 0.867
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A joined REQUIRED transaction ignores its own isolation, timeout and readOnly attributes

## Core idea
A `@Transactional` method with `PROPAGATION_REQUIRED` that is called while a transaction is
already active participates in that transaction instead of starting one. By default the
participating scope takes on the outer transaction's characteristics and silently ignores its
own `isolation`, `timeout` and `readOnly` values. Those attributes take effect when a method
starts a transaction and are ignored when it joins one. Setting `validateExistingTransaction`
to `true` on the transaction manager makes Spring reject the join with an
`IllegalTransactionStateException` when the inner isolation level differs, or when a read-write
scope tries to join a read-only transaction.

## Why choose / why not
- Declare `isolation` and `timeout` on the outermost method, the application-service method that
  owns the use case, when: the setting matters; that method starts the transaction, so it is the
  place where the setting is applied.
- Turn on `validateExistingTransaction` when: inner methods in the codebase declare an isolation
  level or `readOnly = true`, and a mismatch should fail loudly at the call instead of being
  ignored.
- Give the inner method `REQUIRES_NEW` only when: it really needs different settings and may
  commit on its own; it then gets its own settings, at the cost of a second connection.

## Interview angle
- Probed as "I put `SERIALIZABLE` on the inner method; why does it behave like the default?".
- Common wrong answer: "the annotation on the innermost method wins."
- Strong answer: the attributes are applied when a physical transaction begins, and a joining
  method only shares it; then say how the team enforces it, with the validation flag or a rule
  that only service methods carry attributes.

## Related
- [[REQUIRES_NEW keeps the outer connection while it borrows a second one from the pool]]: the
  independent inner transaction is the one way for an inner method to get its own isolation or
  timeout, and that note is its cost.
- [[Catching an exception from a joined REQUIRED method ends in UnexpectedRollbackException]]:
  the same participation rule seen from the rollback side; the joined scope does not own the
  physical transaction, so it can neither set its attributes nor roll it back on its own.
