---
tags: [kubernetes, deployment, deployment-strategy, ops, interview]
status: draft
author: claude
up: ["[[Kubernetes MOC]]"]
source: "https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#recreate-deployment"
created: 2026-10-01
score: 0.886
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A Deployment's Recreate strategy keeps old and new Pods apart only during upgrades

## Core idea
In Kubernetes (v1.37 docs), a Deployment with `.spec.strategy.type: Recreate` kills all existing
Pods before it creates new ones. During an upgrade, all Pods of the old revision are terminated
immediately, and their successful removal is awaited before any Pod of the new revision is created.
The docs state that this guarantees Pod termination before creation only for upgrades. If you
delete a Pod manually, its lifecycle is controlled by the ReplicaSet, and the replacement is created
immediately, even if the old Pod is still in a Terminating state. For an "at most" guarantee on the
number of running Pods, the docs point to a StatefulSet instead.

## Why choose / why not
- Choose Recreate when: two versions must never run side by side during a deploy, such as a single
  writer on a volume that only one Pod can mount, or an on-disk format the old version cannot
  read; accept downtime on every deploy.
- Don't rely on Recreate for an at-most-one guarantee: after a manual delete the replacement starts
  while the old Pod may still be terminating; use a StatefulSet, or a lock in the application.
- Don't choose it for a stateless service: a rolling update keeps serving during the deploy,
  while Recreate always leaves a gap.

## Interview angle
- Probed as "how do you make sure two instances of a single-writer service never overlap on
  Kubernetes?"
- Common wrong answer: "`replicas: 1` with `Recreate`, so it can never run twice."
- Strong answer: Recreate orders only the upgrade; a directly deleted Pod is replaced at once; for
  at most one running Pod use a StatefulSet, and protect the data with a lock or lease anyway.

## Related
- [[A Deployment rolling update is bounded by maxSurge and maxUnavailable]]: that note names
  Recreate as the other Deployment strategy; this one covers the limit of its guarantee.
- [[A StatefulSet gives each pod a stable identity and its own PersistentVolumeClaim]]: the
  controller that the docs point to when you need an at-most-one guarantee.
