---
tags: [system-design, interview, scaling, stateless, cloud]
status: draft
author: claude
source: "https://learn.microsoft.com/en-us/azure/architecture/guide/design-principles/scale-out"
created: 2026-09-30
score: 0.869
review: "borderline"
score_reasons: ["atomic: 0.53 (borderline)"]
judged_by: "jev-1.13.0"
judge_rounds: 2
---
# Stateless services scale out, while scaling up one machine stops at a hardware limit

## Core idea
There are two ways to add capacity to a service. Scaling up (vertical scaling) moves it to a bigger
machine; that often makes the system temporarily unavailable while it is redeployed, and a single
server eventually reaches a limit where memory and processors can't grow any further. Scaling out
(horizontal scaling) adds or removes instances while the application keeps running, so it is not
bound by one machine's size, but only if any instance can handle any request. That is why scaling
out needs stateless services: session state kept in memory causes session affinity, which routes a
client to the same server every time and limits scale-out.

## Why choose / why not
- Scale out when: the service keeps no per-client state in its own memory and load varies, so
  instances can be added and removed with demand.
- Scale up when: the component is stateful and hard to split, such as a single relational primary,
  and a bigger machine still has headroom; accept the brief outage of the resize.
- Move session state to a shared store first when: the service keeps sessions in memory; until
  then, extra instances only pin clients to particular servers.

## Interview angle
- Probed as "how would this service take ten times the traffic?"; the follow-up is "where is the
  state?", because in-memory sessions decide whether scaling out works at all.
- Common wrong answer: "add more pods", without checking the database behind them.
- Strong answer: stateless instances behind a load balancer, state moved to a shared store, and the
  database named as the next limit, with replicas or sharding as its answers.

## Related
- [[A Deployment treats its pods as interchangeable replicas of a stateless workload]]: the
  Kubernetes form of this idea; a Deployment can treat pods as interchangeable only because the
  service keeps no state in them.
- [[A write-heavy database scales by sharding, because replicas add only read capacity]]: once the
  stateless tier scales out, the database is usually the next bottleneck, and that note covers how
  it scales.
- [[System design MOC]]: the scaling entry of the System design cluster, for the question
  of what gives way first as traffic grows.
