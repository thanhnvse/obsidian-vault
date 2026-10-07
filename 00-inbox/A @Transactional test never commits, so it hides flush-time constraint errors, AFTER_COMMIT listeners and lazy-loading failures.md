---
tags: [java, spring, testing, transactions, interview]
status: draft
author: claude
up: ["[[@Transactional MOC]]"]
source: "https://docs.spring.io/spring-boot/docs/3.2.1/reference/html/features.html#features.testing.spring-boot-applications"
created: 2026-10-07
review: unjudged
---
# A @Transactional test never commits, so it hides flush-time constraint errors, AFTER_COMMIT listeners and lazy-loading failures

## Core idea
`@DataJpaTest` and any test annotated `@Transactional` run inside a transaction that the test
framework opens before the test and rolls back afterwards. The service's `REQUIRED` methods join it,
so the service's own commit never happens, and what a commit does is exactly what the test no longer
sees. In a lab on Boot 3.2.1 a duplicate on a unique column was reported only by `flush()`, not by
`save` (the sequence-generated id defers the `INSERT`); an `AFTER_COMMIT` listener never ran; and a
lazy association loaded after the service returned because the test keeps the session open, where a
test without a transaction got `LazyInitializationException`. A `RANDOM_PORT` server runs on another
thread in its own transaction, so its writes survive the test's rollback.

## Why choose / why not
- Leave the test non-transactional when: the use case has constraints, listeners or lazy loading to
  prove; it then runs as in production, at the price of cleanup (truncate in `@AfterEach`, or one
  schema per context).
- Call `entityManager.flush()` and `clear()` inside the transaction when: you only need the real SQL
  and a real read; there is still no commit, no listener, and the session stays open for lazy loads.
- Use `TestTransaction.flagForCommit()` then `end()`, or `@Commit`, when: a listener must run in a
  transactional test; the test must then clean up what it committed.
- Use a real database (Testcontainers) when: isolation, locking or SQL dialect matter; it needs
  Docker.

## Interview angle
- Probed as "my `@DataJpaTest` passes and the code fails in production".
- Common wrong answer: "`@Transactional` on the test keeps it clean, so it is the safe default."
- Strong answer: a never-committing transaction hides flush-time constraint errors, `AFTER_COMMIT`
  listeners and lazy-loading failures; offer non-transactional tests with cleanup, or
  Testcontainers, and say the test-managed rollback does not cover a server running on another
  thread.

## Related
- [[@Transactional MOC]]: the production side of this note: what a commit does, which is exactly
  what a rolled-back test never shows.
- [[@TransactionalEventListener moves a side effect after the commit but loses it if the process dies]]:
  how an `AFTER_COMMIT` listener behaves in production; this note is about why a test that rolls
  back never fires it.
- [[Work handed to another thread runs outside the caller's Spring transaction]]: the same
  thread-bound transaction explains why a `RANDOM_PORT` server's writes are not rolled back by the
  test.

Written up in win-interview: backend/java/docs/testing-strategy.md, section 2.6
