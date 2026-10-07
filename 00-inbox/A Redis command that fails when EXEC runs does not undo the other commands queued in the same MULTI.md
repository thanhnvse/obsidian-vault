---
tags: [redis, transactions, atomicity, interview]
status: draft
author: claude
up: ["[[Caching MOC]]", "[[Concurrency MOC]]"]
source: "https://redis.io/docs/latest/develop/using-commands/transactions/"
created: 2026-10-07
review: unjudged
---
# A Redis command that fails when EXEC runs does not undo the other commands queued in the same MULTI

## Core idea
`MULTI` queues commands and `EXEC` runs them in a row with no other client's request served in
between, so the batch is isolated. It is not all-or-nothing, because
Redis has no rollback: if a queued command fails at run time, `EXEC` puts the error at that position
in its reply array and the other commands stay applied. In the write-up's test, `SET first`, `INCR`
on a non-number and `SET second` returned `OK`, an error and `OK`, and both keys were set. A command
that cannot even be queued, such as `INCR` with three arguments, is different: `EXEC` answers
`EXECABORT` and nothing runs. A Lua script behaves the same way: a script that fails halfway keeps
the writes it already made.

## Why choose / why not
- Use `MULTI`/`EXEC` when: a batch of writes must run with nobody in the middle and each command is
  known to succeed, such as `SET`s of the right type.
- Add `WATCH` when: the write depends on a value you read first; `EXEC` returns nil and runs nothing
  if a watched key changed, so you retry in a loop.
- Use a Lua script when: read, decide and write must be one atomic step; it blocks every other
  client while it runs and does not roll back either.
- Don't use it when: you need all-or-nothing across commands that can fail at run time; replies come
  back only at `EXEC`, so a later command cannot use an earlier result. Keep that in a database
  transaction.
- Give each `WATCH`/`MULTI` sequence its own connection: a Lettuce connection is shared by all
  threads, and a command from another thread is queued into your open transaction.

## Interview angle
- Probed as "Is Redis atomic: `MULTI` or Lua?" and "what does `EXEC` return when one command inside
  fails?"
- Common wrong answer: "`MULTI`/`EXEC` is a transaction, so it rolls back on error", and the same
  for Lua.
- Strong answer: each command is atomic; `MULTI` isolates a batch but has no rollback, and only an
  error found while queuing aborts it; `WATCH` gives compare-and-set with a retry; a script does
  read-decide-write; on Redis 8.4 and later one `SET ... IFEQ` is a compare-and-set.

## Related
- [[Caching MOC]]: the Redis map; this note is the transaction rule behind updating a shared counter
  or stock gate safely.
- [[Concurrency MOC]]: the same "a check, then a write, is two steps" problem as the database
  lost-update notes.
- [[A version column detects a lost update at write time instead of blocking the other writer]]:
  `WATCH` is the same optimistic idea, a version check applied to a Redis key.
- [[A conditional UPDATE with the stock check in its WHERE clause cannot oversell under READ COMMITTED]]:
  the database form of check and write in one step; in Redis the equivalent is one command or a Lua
  script (in the write-up's test, 40 buyers and 10 in stock gave exactly 10 sales).
- Written up in win-interview: backend/docs/redis.md, section 2.3
