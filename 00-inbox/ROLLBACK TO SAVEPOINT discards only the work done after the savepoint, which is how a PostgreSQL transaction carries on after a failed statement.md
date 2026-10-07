---
tags: [database, postgresql, transactions, savepoint, interview]
status: draft
author: claude
up: ["[[Database MOC]]"]
source: "https://www.postgresql.org/docs/15/tutorial-transactions.html"
created: 2026-10-07
review: unjudged
---
# ROLLBACK TO SAVEPOINT discards only the work done after the savepoint, which is how a PostgreSQL transaction carries on after a failed statement

## Core idea
In PostgreSQL an error in a statement aborts the whole transaction: the next statement, even
`SELECT 1`, fails with SQLSTATE `25P02` until the transaction ends. A `COMMIT` then commits nothing,
and with pgjdbc 42.7.1 it returns normally, so a service that catches the exception, carries on and
commits "succeeds" with none of its writes. A savepoint is a mark inside the transaction. Rolling
back to it discards the changes made after it and keeps the earlier ones; in the lab the transaction
was usable again afterwards. pgjdbc can set the savepoints for you: `autosave=always` sets one before
each query and rolls back to it on failure, so a failed statement does not abort the transaction. The
default is `never`.

## Why choose / why not
- Use a savepoint when: one bad row must not abort a batch; the caller must then decide what the
  kept partial work means.
- Validate first, or use `INSERT ... ON CONFLICT`, when: the failure is expected, such as a
  duplicate key; there is then no failure to recover from.
- Don't turn on `autosave=always` everywhere: it sets a savepoint for every statement, a cost paid
  even when nothing fails.
- Roll back and retry the whole transaction when: the caller cannot say what partial work should be
  kept; a savepoint leaves that decision to the code that catches the error.

## Interview angle
- Asked as "a statement in my transaction failed; what state is it in?", or "I caught the exception
  and committed, so why are the rows missing?".
- Common wrong answer: "a failed statement just fails and the rest carries on", or "the commit
  returned, so the data is saved".
- Strong answer: the transaction is aborted, every statement fails with `25P02`, `COMMIT` becomes a
  rollback and the driver may not throw; a savepoint (or driver autosave) is the way to continue;
  log the SQLSTATE where you catch.

## Related
- [[Database MOC]]: this is the failure side of the all-or-nothing guarantee, and the tool that lets a
  transaction keep part of its work after an error.
- [[A serialization failure must be retried as a new transaction that re-runs its reads]]: names the
  same aborted state for an error that must be retried from the start; this note is the other exit,
  for an error you can undo in place.
- [[A deadlock aborts the whole transaction in PostgreSQL but only one statement in Oracle]]: how much
  of the transaction an error takes with it differs by database.
- [[Catching an exception from a joined REQUIRED method ends in UnexpectedRollbackException]]: the
  Spring-level version of the same trap; catching the exception does not save the transaction, but
  Spring's rollback-only mark makes the commit throw where pgjdbc stays quiet.

Written up in win-interview: backend/docs/acid.md, section 2.1 (and the savepoint rows of section 3)
