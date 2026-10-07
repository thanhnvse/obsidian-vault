---
tags: [system-design, chat, ordering, messaging, interview]
status: draft
author: claude
up: ["[[System design MOC]]", "[[Microservices and messaging MOC]]"]
source: ""
created: 2026-10-07
review: unjudged
---
# A gap-free per-conversation sequence number lets a chat client detect a lost message, which timestamps and database sequence ids cannot

## Core idea
Chat needs order inside one conversation, not a global order. Server timestamps fail because clocks
on different nodes disagree and two messages can share a millisecond; a global `bigserial` id is taken
at insert, not at commit, and rollbacks leave gaps. The write-up keeps `last_seq` on the conversation
row: one transaction runs `select last_seq ... for update`, inserts the message with `last_seq + 1`
and updates the counter. Concurrent senders get consecutive (liên tiếp) numbers 1 to N, a sender in
another conversation does not wait, and a rolled-back send leaves no gap, so a missing `seq` means a
lost message. The client keeps the last applied `seq`, applies `n + 1` only after `n`, ignores a `seq`
at or below it and, after a reconnect, asks for `afterSeq=<n>`; deliveries 3, 1, 1, 2, 5, 3, 4, 5 are
applied as 1 to 5, once each.

## Why choose / why not
- Choose it when: messages need a per-conversation order and at-least-once delivery, with the
  receiver deduplicating by `seq`; Kafka keyed by conversation id carries the delivery.
- Don't rely on Kafka partition order alone: it orders delivery but numbers nothing, so a client
  cannot detect a gap or a duplicate from it.
- Don't choose it for a channel with tens of thousands of messages per second: one row lock per
  conversation per message becomes the limit; shard the counters with a merge, or keep an append-only
  log per channel.
- Don't order by timestamp or by a global id: clocks disagree, and neither shows a gap.
- Partition by conversation id: it gives order, and a hot group is then capped at one consumer's
  throughput.

## Interview angle
- Probed as "Design chat. How is the order guaranteed?"
- Common wrong answers: "order by timestamp", "a global auto-increment id", or "exactly-once delivery".
- Strong answer: a counter per conversation on its row (consecutive, no gap on rollback), Kafka keyed
  by conversation for delivery, a client that applies by `seq`, drops duplicates and syncs by
  `afterSeq`; at-least-once plus an idempotent receiver is the honest design.

## Related
- [[System design MOC]]: ordering and fan-out are the deep dives of the chat design.
- [[Microservices and messaging MOC]]: delivery that can repeat is the "sees the message twice"
  question; the `seq` makes the repeat harmless.
- [[A PostgreSQL sequence never reuses a value taken by a rolled-back transaction, so identity keys have gaps]]:
  partial overlap; that note says a gapless number needs a counter row updated in the business
  transaction, which serialises its writers, and this note applies that trade to chat, where the
  gap-free property is what makes a missing `seq` mean a lost message.
- [[Kafka orders records only within a partition, and the record key chooses the partition]]: keying
  by conversation id carries the per-conversation order in delivery.
- [[Committing Kafka offsets after processing gives at-least-once delivery, so consumers must be idempotent]]:
  why the receiver deduplicates by `seq`.
- Written up in win-interview: backend/docs/system-design-walkthrough.md, sections 6.4 and 6.5
