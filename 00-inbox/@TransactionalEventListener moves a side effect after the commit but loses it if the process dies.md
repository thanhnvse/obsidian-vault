---
tags: [java, spring, transactions, events, messaging, interview]
status: draft
author: claude
source: "https://docs.spring.io/spring-framework/reference/data-access/transaction/event.html"
created: 2026-09-30
score: 0.817
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# @TransactionalEventListener moves a side effect after the commit but loses it if the process dies

## Core idea
`@TransactionalEventListener` binds an event listener to a phase of the transaction in which the
event was published, `AFTER_COMMIT` by default, so a side effect such as an email or a Kafka send
runs only after a successful commit and never for work that rolled back. With no transaction
running, the listener is not invoked at all unless `fallbackExecution` is set. The listener runs
in the same process after the commit, so a crash between the commit and the listener loses the
side effect.

## Why choose / why not
- Choose an `AFTER_COMMIT` listener for best-effort effects that may occasionally be lost, such
  as cache eviction or a confirmation email: it is in-process, simple, and never announces work
  that rolled back.
- Choose a transactional outbox instead for effects that must not be lost, such as integration
  events: the message is stored in the same transaction as the business data and a separate
  relay publishes it. The relay can publish twice, so consumers must be idempotent.
- If the listener writes to the database, run that write with `REQUIRES_NEW`; otherwise it joins
  the finished transaction and is never committed.

## Interview angle
- Probed as "how do you send the confirmation only if the order really committed?", then "and if
  the process dies right after the commit?".
- Common wrong answer: "publish from `AFTER_COMMIT` and it is reliable"; it delivers at most
  once, while an outbox delivers at least once.
- Strong answer: `AFTER_COMMIT` fixes "announced but rolled back", not "committed but never
  announced"; choose it or an outbox by asking whether losing the effect is acceptable.

## Related
- [[REQUIRES_NEW keeps the outer connection while it borrows a second one from the pool]]: a
  write inside an `AFTER_COMMIT` listener needs `REQUIRES_NEW`, and the finished transaction's
  resources may still be bound at that point, so the write can take a second connection.
- [[A checked exception commits a Spring @Transactional method by default]]: that silent commit
  also fires `AFTER_COMMIT` listeners, so a caller that sees the exception may find the side
  effect already sent.
