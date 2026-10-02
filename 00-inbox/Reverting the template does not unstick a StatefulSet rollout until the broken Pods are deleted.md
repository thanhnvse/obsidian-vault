---
tags: [kubernetes, statefulset, rollback, ops, interview]
status: draft
author: claude
up: ["[[Kubernetes MOC]]"]
source: "https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#forced-rollback"
created: 2026-10-01
score: 0.893
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Reverting the template does not unstick a StatefulSet rollout until the broken Pods are deleted

## Core idea
In Kubernetes (v1.37 docs), a StatefulSet that uses the `RollingUpdate` strategy with the default
`OrderedReady` Pod management policy updates its Pods one at a time, and it waits for each updated
Pod to be Running and Ready before it updates the next one. If the new Pod template produces a Pod
that never becomes Running and Ready, for example because of a bad binary or an application-level
configuration error, the StatefulSet stops the rollout and waits. Reverting the Pod template to a
good configuration is not enough on its own: because of a known issue, the StatefulSet keeps
waiting for the broken Pod to become Ready before it will try to revert it. After reverting the
template, you must also delete every Pod that the StatefulSet had already tried to run with the
bad configuration, and the StatefulSet then recreates those Pods from the reverted template.

## Why choose / why not
- Revert and then delete the broken Pods when: an `OrderedReady` rollout is stuck on a Pod that
  is not Ready; delete only the Pods created from the bad template, not the healthy ones.
- Don't expect `kubectl rollout undo statefulset/<name>` alone to finish the job: it reverts the
  template, but the Pod that never became Ready still blocks the rollout until you delete it.
- Choose `Parallel` Pod management only when: the application tolerates Pods becoming ready out
  of order; it changes how the rollout proceeds, but it is not a fix for a bad template.

## Interview angle
- Probed as "the StatefulSet rollout is stuck, you reverted the image, and it is still stuck.
  Why?"
- Common wrong answer: "reverting the template fixes a stuck StatefulSet rollout."
- Strong answer: find the Pod that is not Ready, revert the template, delete the Pods made from
  the bad template, and name the documented known issue with `OrderedReady` rolling updates.

## Related
- [[Kubernetes reports a stalled Deployment rollout but never rolls it back by itself]]: the
  Deployment side of a stuck rollout; that note covers who must trigger the rollback, and this one
  covers the extra manual step a StatefulSet needs after the revert.
- [[A StatefulSet gives each pod a stable identity and its own PersistentVolumeClaim]]: the
  ordered, one-Pod-at-a-time policy described there is what lets one broken Pod block the whole
  rollout.
