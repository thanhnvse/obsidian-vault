---
tags: [microservices, messaging, outbox, postgresql, interview]
status: draft
author: claude
up: ["[[Microservices and messaging MOC]]"]
source: ""
created: 2026-10-07
review: unjudged
---
# An outbox poller that remembers the highest id it has seen skips a row that commits late, because sequence ids are taken at insert, not at commit

## Core idea
A `bigserial` id comes from `nextval()` when the row is inserted, not when the transaction commits. A
transaction that took id 1 and commits late becomes visible after id 2. A poller that remembers the
highest id it has seen reads id 2 and remembers 2; when id 1 commits it asks for `id > 2` and never
sees it, so the event is skipped forever. A relay that selects `WHERE sent_at IS NULL` finds the late
row, which is why the outbox has a status column instead of a cursor. A partial index
`on outbox (id) where sent_at is null` keeps that query fast while sent rows pile up. The lab proved
the skip with two transactions that commit in the opposite order of their ids.

## Why choose / why not
- Select by status when: a polling relay reads the outbox, with `WHERE sent_at IS NULL ORDER BY id
  LIMIT n` and the partial index above; a row is found whenever it becomes visible.
- Don't keep a cursor on the id when: ids come from a sequence and writers commit concurrently; a
  late commit falls behind the cursor and nothing ever moves it back.
- Choose CDC instead when: you want commit order without a status column; logical decoding hands out
  concurrent transactions in commit order, so there is no id order that can disagree with it.
- Pay for the status column: every mark is an `UPDATE` that leaves a dead row version for `VACUUM`,
  so delete or partition old rows, and alert on the age of the oldest unsent row.

## Interview angle
- Asked as "how does your relay know which outbox rows to publish next?", or "can the poller just
  remember the last id?".
- Common wrong answer: "the relay remembers the last id it published".
- Strong answer: ids are taken at insert and become visible at commit, so a cursor skips late
  commits; select unsent rows by status, or read the log in commit order, and watch the oldest unsent
  row.

## Related
- [[A transactional outbox sends a message if and only if the database transaction commits, but its relay can send it twice]]:
  that note introduces the polling relay; this is a trap in how the relay finds its rows.
- [[A PostgreSQL sequence never reuses a value taken by a rolled-back transaction, so identity keys have gaps]]:
  the same `nextval` property seen from the other side; the value belongs to the insert, not to the
  commit, so it leaves gaps on rollback and arrives out of commit order.
- [[Microservices and messaging MOC]]: the map for "what happens when the other side is slow, down,
  or sees the message twice"; this note keeps the sender from silently losing the message.

Written up in win-interview: backend/docs/idempotency-and-outbox.md, sections 2.4 to 2.6
