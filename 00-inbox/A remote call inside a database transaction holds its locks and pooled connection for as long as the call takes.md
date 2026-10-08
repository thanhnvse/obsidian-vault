---
tags: [database, transactions, performance, connection-pool, interview]
status: draft
author: claude
up: ["[[Database MOC]]"]
source: "https://www.postgresql.org/docs/current/explicit-locking.html"
created: 2026-10-08
review: "unjudged"
score_reasons: ["jev unavailable: HTTP 401 after 1 attempt(s): {\"detail\":{\"error_type\":\"authentication_error\",\"message\":\"Cannot authenticate with the server. Please check your API key and try again.\"}}"]
judge_rounds: 1
---
# A remote call inside a database transaction holds its locks and pooled connection for as long as the call takes

## Core idea
Row locks taken by `UPDATE`, `DELETE` or `SELECT ... FOR UPDATE` are held until the transaction
commits or rolls back, and the transaction keeps its connection checked out of the pool the whole
time. An HTTP or LLM call made between the first write and the commit therefore stretches both:
every other writer to those rows waits for the remote call, and the pool is one connection short
for the call's full duration. In a batch that makes one remote call per row inside one transaction,
the lock time grows with the batch size; a client timeout or a fallback caps each call, not the sum.

## Why choose / why not
- Make the remote call before the transaction when: its result is an input to the write; fetch
  first, then open the transaction and write.
- Make it after the commit, or through an outbox, when: it is a side effect of the write, such as a
  notification or an enrichment that can arrive later.
- Commit one unit of work at a time when: a batch processes many rows, so each row's locks are
  released as soon as that row is done.
- Keep the call inside only when: nothing else contends for the rows the transaction has locked,
  and you have measured the lock wait it causes.

## Interview angle
- Asked as "why is it bad to call another service inside `@Transactional`?" or "why did the
  connection pool run dry when a partner API slowed down?".
- Common wrong answer: "it is fine because the call has a timeout".
- Strong answer: locks and the connection are held until commit, so remote latency becomes other
  writers' lock wait and pool starvation; move the call out of the transaction or use an outbox.

## Related
- [[A @Transactional timeout fails the next database access after the deadline instead of interrupting the method]]:
  a transaction timeout does not stop the remote call either; it only fails the next database
  access after the deadline.
- [[A transactional outbox sends a message if and only if the database transaction commits, but its relay can send it twice]]:
  the standard way to move a side effect out of the transaction without losing it.
- [[Database MOC]]: the map entry for what an open transaction holds.
- Seen in: LEO-CDP/leo-customer360, customer360-backend/identity_resolution/identity_resolution/resolver.py,
  which asks an LLM to name a persona for each hashed profile inside the loop of one batch
  transaction that commits only after the loop (read 2026-10-08).
