---
tags: [system-design, messaging, autoscaling, interview]
status: draft
author: claude
up: ["[[System design MOC]]"]
source: "https://learn.microsoft.com/en-us/azure/architecture/best-practices/auto-scaling"
created: 2026-10-01
score: 0.89
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Scale queue workers on the age of the oldest message, not on queue depth

## Core idea
Queue length is a usable autoscaling signal, but it says nothing about how fast the queue drains or how urgent its messages are. Microsoft's autoscaling guidance names a better attribute: critical time, the time between when a message was sent and when its processing was complete. If critical time is within the acceptable business range, scaling out is unnecessary even when the queue is long. Its example contrasts 50,000 queued messages for partner email integration whose oldest has a 500 ms critical time, which may not justify the cost of scaling, with 500 messages at the same 500 ms in a real-time game with a 100 ms target, where scaling out makes sense.

## Why choose / why not
- Scale on critical time when: consumers serve a latency target; it tracks the delay users actually feel.
- Scale on queue length when: messages carry no send timestamp; it is a rough proxy that over-scales tolerant work and under-scales urgent work.
- Always cap the instance count when: consumers write to a database; the guidance recommends a maximum so the backlog does not simply move to the next bottleneck.

## Interview angle
- Probed as "which metric would you autoscale queue workers on?"
- Common wrong answer: "queue depth above N" with no word on drain rate or urgency.
- Strong answer: critical time per queue against its own target, plus a maximum instance count.

## Related
- [[A queue in front of a service levels load spikes at the cost of an immediate response]]: that note is why the queue exists; this one is how to size the consumers behind it.
