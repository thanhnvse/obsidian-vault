---
tags: [kubernetes, deployment, ops, interview]
status: draft
author: claude
source: "https://kubernetes.io/docs/concepts/workloads/controllers/deployment/"
created: 2026-09-30
score: 0.908
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A Deployment rolling update is bounded by maxSurge and maxUnavailable

## Core idea
In Kubernetes (v1.37 docs), `RollingUpdate` is the default Deployment strategy; the only other
type, `Recreate`, kills all existing Pods before it creates new ones. During a rolling update,
`maxUnavailable` caps how many Pods may be unavailable and `maxSurge` caps how many may be
created above the desired count. Both accept an absolute number or a percentage and default to
25%; a percentage is rounded down for `maxUnavailable` and up for `maxSurge`, and
`maxUnavailable` cannot be 0 when `maxSurge` is 0. A new Pod counts as available only after it
has been ready for `minReadySeconds`, which defaults to 0, so readiness sets the pace of the
rollout.

## Why choose / why not
- Set `maxUnavailable: 0` and `maxSurge: 1` when: the service already runs near its capacity and
  must never lose a replica; the cost is one extra Pod's resources and a slower rollout.
- Raise `maxUnavailable` or `maxSurge` when: the cluster has headroom and a faster rollout
  matters more than keeping every replica up.
- Use `Recreate` instead when: two versions must never run at the same time, for example a
  single volume that only one Pod can mount; accept a short outage on every deploy.

## Interview angle
- Probed as "how do you deploy without downtime on Kubernetes?"; a rolling update plus a
  readiness probe that only passes once the application can serve traffic.
- Common wrong answer: "RollingUpdate is zero-downtime by itself." Traffic follows readiness, so
  a probe that passes too early sends requests to a JVM that is still starting.
- Strong answer: do the arithmetic. With 3 replicas and the defaults, `maxSurge` rounds up to 1
  and `maxUnavailable` rounds down to 0, so the rollout never drops below 3 ready Pods.

## Related
- [[A Deployment treats its pods as interchangeable replicas of a stateless workload]]: that note
  explains the new ReplicaSet per template change; this one covers the rate at which the
  Deployment swaps Pods from the old ReplicaSet to the new one.
- [[Kubernetes MOC]]: the Ops and cloud cluster asks how a service ships, and this is
  the default answer.
