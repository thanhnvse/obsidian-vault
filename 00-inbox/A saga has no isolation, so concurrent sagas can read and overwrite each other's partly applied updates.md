---
tags: [microservices, saga, consistency, concurrency, interview]
status: draft
author: claude
up: ["[[Microservices and messaging MOC]]"]
source: "https://learn.microsoft.com/en-us/azure/architecture/patterns/saga"
created: 2026-10-01
score: 0.907
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A saga has no isolation, so concurrent sagas can read and overwrite each other's partly applied updates

## Core idea
A saga runs a business transaction as a sequence of local transactions, and each one commits in its
own service before the next one starts. Its intermediate state is therefore real, committed data
that other sagas and requests can see while the saga is still running. The Azure Architecture
Center's Saga page says that because each service manages its own data, there's no built-in
isolation across services, and names the typical anomalies: lost updates, when one saga modifies
data without considering changes made by another saga; dirty reads, when a saga reads data that
another saga has modified but not yet finished; and fuzzy, or nonrepeatable, reads, when different
steps of one saga read inconsistent data because updates occur between the reads. The application
has to build the missing isolation itself, and the page lists countermeasures: a semantic lock, an
application-level flag such as a "pending" status that marks an update in progress; commutative
updates that give the same result in any order; a pessimistic view that reorders the saga so data
updates occur in retryable transactions; rereading values to confirm they are unchanged before
updating; and version files that log the operations on a record so they run in the correct order.

## Why choose / why not
- Choose a saga when: the steps live in services that each own their database, a distributed
  transaction is not available, and the business can show intermediate states for a while, such
  as an order marked "pending" until payment and stock are confirmed.
- Add a countermeasure before shipping when: another flow could act on a half-finished state, such
  as a second order reading stock that a failing saga will release again; a "pending" status as a
  semantic lock, or a reread before the update, closes that gap.
- Don't choose a saga when: the operations must stay isolated from each other, such as two
  transfers that must never see each other's partial balance; keep that data in one service and
  one ACID transaction.

## Interview angle
- Probed as "two customers book the last seat through a saga at the same time; what can go wrong?"
  Each saga sees the other's committed steps, so without a countermeasure both can proceed, one
  update can overwrite the other, and one saga has to be compensated after the fact.
- Common wrong answer: "a saga is a distributed transaction, and the compensations roll it back."
  It has no isolation, and compensations are new local transactions that run after others may have
  read the data; the same page warns they might not always succeed.
- Strong answer: name the missing I of ACID, one anomaly with a concrete example, and one
  countermeasure, such as a "pending" status that other sagas respect.

## Related
- [[Choreography spreads a saga's flow across event subscriptions, while orchestration keeps it in one coordinator]]:
  that note decides where a saga's sequence of steps lives; this note covers what neither style
  provides, isolation between concurrent sagas, so its countermeasures apply to both.
- [[A version column detects a lost update at write time instead of blocking the other writer]]:
  the saga page's "reread values" countermeasure is the same check across services: confirm the
  data is unchanged before writing, and restart when it changed.
