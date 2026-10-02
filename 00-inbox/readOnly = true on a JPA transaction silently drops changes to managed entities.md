---
tags: [java, spring, transactions, jpa, hibernate, interview]
status: draft
author: claude
source: "https://docs.spring.io/spring-data/jpa/reference/jpa/transactions.html"
created: 2026-09-30
score: 0.787
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# readOnly = true on a JPA transaction silently drops changes to managed entities

## Core idea
Inside a read-write `@Transactional` method, Hibernate detects a change to a managed entity and
writes it when the persistence context is flushed at commit, with no `save()` call. With
`readOnly = true`, Spring sets the Hibernate flush mode to `MANUAL`, which makes Hibernate skip
dirty checks, so the same change is never written and no error is raised. The flag is an
optimisation hint that is also passed to the JDBC driver; Spring's `TransactionDefinition`
states that it does not necessarily make write attempts fail. The Spring Data JPA 4.1 reference
documents the flush mode, and a lab on Spring Framework 6.1.2 with Hibernate 6.4.1 confirmed the
silent drop.

## Why choose / why not
- Choose `readOnly = true` for query-only service methods: Hibernate skips dirty checking, which
  helps most on large object graphs, and a routing `DataSource` can send the work to a replica by
  checking `TransactionSynchronizationManager.isCurrentTransactionReadOnly()`.
- Don't use it as a guard against writes: a change to an entity disappears without an error, and
  whether a native `UPDATE` is rejected depends on the driver and the database. Enforce read-only
  access with a database user or replica that has no write rights.
- Don't leave it on a method that changes entities, such as a writing method without its own
  `@Transactional` in a class annotated `@Transactional(readOnly = true)`; the change is lost.

## Interview angle
- Probed as "what does `readOnly = true` actually do?", often after "do I need `save()` after
  changing an entity?".
- Common wrong answer: "`readOnly` prevents writes."
- Strong answer: no dirty checking and no flush, a read-only hint to the driver, possible replica
  routing; not a guard, because a change made inside it is dropped rather than rejected.

## Related
- [[A joined REQUIRED transaction ignores its own isolation, timeout and readOnly attributes]]:
  the flag follows the transaction that started first; a read-only method that joins a
  read-write transaction loses nothing, while a read-write method that joins a read-only one
  loses its changes.
