---
tags: [kubernetes, deployment-strategy, ops, interview]
status: draft
author: claude
source: "https://kubernetes.io/docs/concepts/workloads/management/"
created: 2026-09-30
score: 0.921
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A Kubernetes canary built from two Deployments splits traffic by replica count

## Core idea
In Kubernetes (v1.37 docs), a canary runs the new release side by side with the previous one as a
second Deployment, so the new release receives live production traffic before it is fully
rolled out. The two Deployments share their common labels and differ in one label, such as
`track: stable` and `track: canary`. The Service selects only the common labels, leaving out
`track`, so it sends traffic to the Pods of both Deployments. The ratio of stable to canary
replicas, such as 3:1, sets each release's share of the live traffic. Once you are confident,
you update the stable Deployment to the new release and remove the canary one.

## Why choose / why not
- Choose when: you want a canary built from plain Kubernetes objects only, and a traffic share
  that moves in whole-Pod steps is precise enough.
- Don't choose when: you need a small exact share, since 1% takes 99 stable Pods for 1 canary, or
  you must pick which users see the canary; use an ingress controller or a service mesh that
  routes by weight or by header instead.
- Watch for: both Deployments serve production at once, so each needs its own readiness probe,
  resources and dashboards split by the `track` label.

## Interview angle
- Probed as "how would you run a canary on plain Kubernetes?"; two Deployments, one shared
  Service, and the replica ratio as the dial.
- Common wrong answer: "tell the Service to send 10% to the canary." A Service has no weight per
  Deployment; the share follows how many ready Pods each Deployment puts behind it.
- Strong answer: add that the split is per connection, not per user, unless session affinity is
  set, so one user can reach both versions and both must work against the same schema.

## Related
- [[Blue-green switches all traffic at once while a canary shifts a subset of users first]]:
  that note compares the canary idea with blue-green; this one covers how to build the canary
  from Kubernetes objects.
- [[Expand and contract schema changes keep the previous version runnable after a rollback]]:
  both tracks read and write one database at the same time, so the schema must suit both.
