---
tags: [kubernetes, deployment, ops, interview]
status: draft
author: claude
up: ["[[Kubernetes MOC]]"]
source: "https://kubernetes.io/docs/concepts/workloads/pods/disruptions/"
created: 2026-10-01
score: 0.863
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A PodDisruptionBudget does not limit a Deployment or StatefulSet rolling update

## Core idea
In Kubernetes (v1.37 docs), a PodDisruptionBudget (PDB) limits how many Pods of a replicated
application can be down at the same time from voluntary disruptions, such as a node drain that
evicts Pods through the Eviction API. Pods that are deleted or unavailable because of a rolling
upgrade count against the budget, but workload resources such as Deployments and StatefulSets are
not limited by PDBs when they do rolling upgrades. Instead, the handling of failures during an
application update is configured in the spec of the workload resource itself, which for a
Deployment means `maxUnavailable` and `maxSurge`. Deleting Deployments or Pods directly also
bypasses PDBs.

## Why choose / why not
- Use a PDB when: nodes are drained for upgrades or by an autoscaler, and the application needs a
  minimum number of running replicas, such as a quorum.
- Use `maxUnavailable` and `maxSurge` when: you want to bound lost capacity during your own
  release; that is where the rollout reads its limits.
- Don't treat a PDB as release protection: a bad rollout can take Pods down that the PDB never
  sees as evictions.

## Interview angle
- Probed as "we have a PDB with minAvailable 2, so a deploy can't take us below 2, right?"
- Common wrong answer: "yes, the PDB applies to every way a Pod goes down."
- Strong answer: a PDB guards voluntary disruptions that go through the Eviction API; rollouts are
  bounded by the workload's update strategy; the two meet because Pods made unavailable by a
  rollout still use up the budget that a node drain checks.

## Related
- [[A Deployment rolling update is bounded by maxSurge and maxUnavailable]]: that bound, not the
  PDB, is what limits a Deployment's rollout; this note explains why the PDB does not.
