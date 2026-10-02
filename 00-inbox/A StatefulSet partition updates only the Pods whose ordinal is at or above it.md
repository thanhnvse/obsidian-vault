---
tags: [kubernetes, statefulset, deployment-strategy, ops, interview]
status: draft
author: claude
up: ["[[Kubernetes MOC]]"]
source: "https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#partitions"
created: 2026-10-01
score: 0.887
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A StatefulSet partition updates only the Pods whose ordinal is at or above it

## Core idea
In Kubernetes (v1.37 docs), a StatefulSet's `RollingUpdate` strategy can be partitioned by setting
`.spec.updateStrategy.rollingUpdate.partition`. When the Pod template changes, every Pod with an
ordinal greater than or equal to the partition is updated, and every Pod with an ordinal less than
the partition is not. A Pod below the partition that is deleted is recreated at the previous
version, so it does not pick up the new template either. If the partition is greater than
`.spec.replicas`, updates to the Pod template are not propagated to any Pod. The docs name staging
an update, rolling out a canary and performing a phased roll-out as the uses of a partition.

## Why choose / why not
- Choose a partition when: you want a canary for a stateful set; set it to `replicas - 1` so only
  the highest ordinal gets the new template, watch that Pod, then lower the partition step by
  step.
- Don't choose it for a Deployment's canary: a Deployment has no ordinals, so its canary is a
  second Deployment behind the same Service.
- Budget for: a manual, multi-step release; every lower partition value is a change someone must
  apply, unless a progressive-delivery controller automates it.

## Interview angle
- Probed as "how do you canary a StatefulSet?"
- Common wrong answer: "a StatefulSet always updates every Pod, so it cannot be canaried."
- Strong answer: name `rollingUpdate.partition`, say which ordinals get the new template and that
  lower ordinals stay on the old one even when deleted, then describe lowering it step by step.

## Related
- [[A Kubernetes canary built from two Deployments splits traffic by replica count]]: the
  Deployment way to canary; this note covers the StatefulSet way, which splits by ordinal inside
  one controller instead of adding a second one.
- [[A StatefulSet gives each pod a stable identity and its own PersistentVolumeClaim]]: the
  ordinals that the partition is compared with come from that identity.
