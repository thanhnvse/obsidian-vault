---
tags: [microservices, messaging, outbox, postgresql, interview]
status: draft
author: claude
up: ["[[Microservices and messaging MOC]]"]
source: ""
created: 2026-10-07
review: unjudged
---
# Parallel outbox relays that claim the oldest unsent rows with SKIP LOCKED can publish one aggregate's events out of order

## Core idea
With `FOR UPDATE SKIP LOCKED`, two relays claim disjoint rows without waiting. That breaks order per
aggregate: relay A claims `OrderPlaced` and stalls (khựng lại), for example in a GC pause or on a slow
broker; relay B skips the locked row, claims the same order's `OrderPaid` and publishes it first.
The lab reproduced exactly this. The fix is to claim only the head of each aggregate: add `AND NOT
EXISTS` (an earlier unsent row of the same aggregate) to the claim, so a row whose predecessor is
unsent is not eligible even while another relay holds that predecessor, and relay B takes a
different aggregate. The aggregate id must also be the broker key, so that the broker keeps the order
the relay hands over.

## Why choose / why not
- Use one active relay when: it keeps up; order per aggregate comes for free, at the cost of one
  process's throughput and a gap during failover.
- Claim only the heads when: several relays are needed and order per aggregate matters; the cost is
  one event per aggregate per cycle and a correlated subquery, so give it an index on
  `(aggregate_id, id) where sent_at is null` (design advice, not measured in the lab).
- Partition the relays by aggregate hash when: you want a simple claim and aggregates emit bursts; the
  cost is rebalancing whenever the number of relays changes.
- Don't add instances with plain `SKIP LOCKED` just to fix lag when: order per aggregate matters; the
  bug appears only under load, after the scale-out.

## Interview angle
- Asked as "you run three relay instances; what can go wrong?".
- Common wrong answer: "just add instances with `SKIP LOCKED`", or "plain `FOR UPDATE` is enough".
- Strong answer: without a lock two relays publish the same rows; plain `FOR UPDATE` makes them queue;
  `SKIP LOCKED` gives disjoint batches but can reorder one aggregate when a relay stalls; then the
  fixes above, and the reminder that a crashed relay's rows come back, so duplicates stay possible.

## Related
- [[SKIP LOCKED lets several workers claim different rows of a job table without waiting]]: that note
  assumes any free row will do; this one is what that assumption costs when rows of one key must keep
  their order.
- [[A transactional outbox sends a message if and only if the database transaction commits, but its relay can send it twice]]:
  the relay being scaled out; its duplicates and this note's reordering are two of the ways a scaled
  relay can hurt consumers.
- [[Kafka orders records only within a partition, and the record key chooses the partition]]: the
  broker end of the same chain; the aggregate id as record key keeps the order that the relay must
  hand over in sequence.
- [[Microservices and messaging MOC]]: the map this belongs to.

Written up in win-interview: backend/docs/idempotency-and-outbox.md, section 2.4
