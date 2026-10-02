---
tags: [java, spring, transactions, connection-pool, interview]
status: draft
author: claude
source: "https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/tx-propagation.html"
created: 2026-09-30
score: 0.83
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# REQUIRES_NEW keeps the outer connection while it borrows a second one from the pool

## Core idea
`PROPAGATION_REQUIRES_NEW` suspends the current transaction and starts an independent physical
transaction, which commits or rolls back on its own and can declare its own isolation, timeout
and read-only settings. The suspended outer transaction keeps its resources bound, including its
database connection, while the inner transaction acquires a new connection. When several
threads each hold an outer connection and wait for an inner one, the pool can run out and the
threads deadlock; with HikariCP they fail once the connection timeout expires. The Spring
reference therefore advises using `REQUIRES_NEW` only when the pool exceeds the number of
concurrent threads by at least one.

## Why choose / why not
- Choose it when: a record must commit whatever happens to the outer transaction, such as an
  audit row for a rejected attempt; no other propagation gives the inner work its own fate.
- Don't use it on a hot path, or with a pool sized to the number of request threads: each call
  holds two connections per thread. If it must stay, size the pool with HikariCP's rule
  `Tn × (Cm − 1) + 1`, which is threads plus one when each thread holds two connections.
- Keep the write in the outer transaction instead (plain `REQUIRED`) when: the record only
  matters if the business operation commits; then one connection is enough.

## Interview angle
- Probed as "REQUIRED vs REQUIRES_NEW?", followed by "what does it cost?".
- Common wrong answer: "REQUIRES_NEW is a nested transaction." It is a separate physical
  transaction on another connection; `NESTED` is the savepoint-based one.
- Strong answer: name the one real use case, then the cost: two connections per thread, a pool
  that deadlocks itself under load, and the sizing rule.

## Related
- [[Catching an exception from a joined REQUIRED method ends in UnexpectedRollbackException]]:
  `REQUIRES_NEW` is the fix named there when the outer work must continue after an inner
  failure; this note is the price of that fix.
