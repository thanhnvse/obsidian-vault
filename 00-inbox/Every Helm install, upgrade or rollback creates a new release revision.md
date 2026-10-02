---
tags: [kubernetes, helm, ops, interview]
status: draft
author: claude
source: "https://helm.sh/docs/intro/using_helm/"
created: 2026-09-30
score: 0.899
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 2
---
# Every Helm install, upgrade or rollback creates a new release revision

## Core idea
In the Helm docs (v4.3.0), a release is one installed instance of a chart in a cluster, built by
merging the chart's templates with a set of values. Every install, upgrade or rollback
increments the release's revision number by 1, starting from 1. `helm history RELEASE` lists
the revisions, and `helm rollback RELEASE [REVISION]` returns the release to one of them; with
the revision omitted or set to 0 it goes back to the previous release. A rollback is recorded as
a new revision, so revision numbers only go up: rolling revision 3 back to 2 creates revision 4.

## Why choose / why not
- Choose when: you want one command to return every resource of a service, such as its
  Deployment, Service and ConfigMap, to a known revision together, across several environments
  that share one chart.
- Don't choose when: a single environment with a few static manifests; plain `kubectl apply` or
  a Kustomize overlay avoids debugging Go templates.
- Don't expect it to undo data: Helm tracks Kubernetes resources, so a database migration that
  the new version already ran stays applied after `helm rollback`.

## Interview angle
- Probed as "a release went bad, what do you run?"; `helm history`, then `helm rollback` to the
  last good revision, and say that the rollback shows up as a new revision.
- Common wrong answer: "the rollback deletes the bad revision and the counter goes back."
- Strong answer: state the scope: one rollback returns every Kubernetes resource the chart owns
  to the chosen revision, it is recorded as the next revision number, and it leaves data alone.

## Related
- [[A Deployment treats its pods as interchangeable replicas of a stateless workload]]: a chart
  for a Java service usually templates this Deployment, so a Helm upgrade or rollback reaches the
  Pods through the Deployment's own rolling update.
- [[Kubernetes MOC]]: this answers the Ops and cloud question of how a service comes
  back when a release breaks.
