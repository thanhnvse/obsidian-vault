---
tags: [kubernetes, deployment, rollback, ops, interview]
status: draft
author: claude
source: "https://kubernetes.io/docs/concepts/workloads/controllers/deployment/"
created: 2026-09-30
score: 0.904
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 2
---
# kubectl rollout undo rolls back only the Deployment's Pod template

## Core idea
In Kubernetes (v1.37 docs), a Deployment revision is created if and only if the Pod template
(`.spec.template`) changes, so scaling the Deployment creates no revision. `kubectl rollout undo
deployment/<name>` returns the Deployment to its previous revision, or to a chosen one with
`--to-revision=<n>`. The docs state that when you roll back to an earlier revision, only the
Deployment's Pod template part is rolled back. Everything outside that template, such as the
content of a ConfigMap, a Service, or the database schema and its data, stays as the failed
release left it.

## Why choose / why not
- Use `rollout undo` when: the bad change sits in the Pod template, such as a new image tag, an
  environment variable or a resource limit; it is the fastest revert available.
- Don't rely on it alone when: the release also changed the content of a ConfigMap, a Service or
  the database schema; none of those is part of the revision, so revert them separately or make
  them backward compatible before the release.
- Prefer `helm rollback` when: the service is installed by a Helm chart, since that reverts every
  resource the chart owns instead of one Deployment's Pod template.

## Interview angle
- Probed as "you ran `rollout undo` and the service is still broken; why?"; the first cause to
  name is a change outside the Pod template.
- Common wrong answer: "`rollout undo` restores the whole previous release."
- Strong answer: list what a revision holds (the Pod template) and what it does not (other
  objects, data, schema), then say the schema must still work with the previous version.

## Related
- [[A Deployment treats its pods as interchangeable replicas of a stateless workload]]: the
  ReplicaSet created per Pod template change is exactly what a revision is, so that note explains
  where the rollback target comes from.
- [[Every Helm install, upgrade or rollback creates a new release revision]]: the Helm rollback
  works at release level across all chart resources, which is the wider scope this command lacks.
