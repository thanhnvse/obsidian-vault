---
tags: [system-design, interview, messaging, queue, scaling]
status: draft
author: claude
source: "https://learn.microsoft.com/en-us/azure/architecture/patterns/queue-based-load-leveling"
created: 2026-09-30
score: 0.923
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A queue in front of a service levels load spikes at the cost of an immediate response

## Core idea
Queue-based load leveling puts a queue between a task and the service it invokes: the task posts a
message, the queue stores it, and the service takes messages off and processes them at its own pace.
A burst of requests then waits in the queue instead of flooding the service, so the service needs
enough instances for the average load rather than the peak. The price is time and the reply: a
queue is one-way, so a task that needs a result needs a separate response mechanism, and if
producers stay faster than consumers on average, the queue keeps growing and latency keeps rising.
Most queue services deliver at least once, so consumers must be idempotent.

## Why choose / why not
- Choose a queue when: load arrives in intermittent spikes that would overwhelm the downstream
  service or database, and the work may finish later, such as sending email or importing files.
- Don't choose it when: the caller needs a low-latency, synchronous answer, or the load is
  predictably low and steady, so the queue only adds moving parts.
- Cap the consumers when: they autoscale on queue depth; unbounded consumers only move the overload
  onto the database behind them.

## Interview angle
- Probed as "a flash sale floods the order service; how do you keep the database alive?"; put a
  queue in front of the writer and size the consumers to what the database can take.
- Common wrong answer: "the queue makes it faster"; it lets the system survive the peak, while each
  request finishes later.
- Strong answer: say what it buys (smoothing, capacity for the average), what it costs (latency, no
  reply, duplicates) and what you watch: queue depth and how long messages wait.

## Related
- [[A synchronous service call couples the caller to the callee's availability and latency]]: that
  note is about failure coupling between two services; this one uses the same queue to absorb load
  spikes rather than outages.
- [[Batching writes raises throughput by paying the per-request overhead once per batch]]: for bursty
  writes that must not be lost, that note's answer is a durable queue, which is this pattern.
- [[System design MOC]]: the System design cluster's answer to what gives way first when a
  spike hits a service that cannot scale with it.
