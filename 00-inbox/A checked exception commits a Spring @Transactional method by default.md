---
tags: [java, spring, transactions, interview]
status: draft
author: claude
source: "https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/rolling-back.html"
created: 2026-09-30
score: 0.919
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 2
---
# A checked exception commits a Spring @Transactional method by default

## Core idea
By default, Spring marks a transaction for rollback only when the method throws a
`RuntimeException` or an `Error`. A checked exception thrown out of a `@Transactional` method
does not trigger a rollback: the transaction commits and the exception still reaches the
caller, so the caller sees a failure while the writes are already durable. A rollback rule such
as `rollbackFor = Exception.class` makes checked exceptions roll back too.

## Why choose / why not
- Keep the default when: the codebase signals failure with unchecked domain exceptions, so every
  failure already rolls back.
- Add `rollbackFor = Exception.class` when: methods declare checked exceptions that mean "the
  operation failed"; put it on every such method, because one missing annotation is a silent
  partial commit.
- Don't add `noRollbackFor` unless: the exception really means the work done so far must be
  kept, such as recording a rejected attempt.

## Interview angle
- Probed as "why didn't my transaction roll back?"; a checked exception is the first cause to
  name, before self-invocation or a swallowed exception.
- Common wrong answer: "`@Transactional` rolls back on any exception."
- Strong answer: state the rule, then the team convention that makes it safe, for example
  unchecked domain exceptions everywhere.

## Related
- [[@Transactional MOC]]: this is the first rollback rule interviewers probe in the
  Spring section, so the map lists it as the entry point for transaction questions.
