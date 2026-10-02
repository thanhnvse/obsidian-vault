---
tags: [kubernetes, deployment, rollback, ops, interview]
status: draft
author: claude
up: ["[[Kubernetes MOC]]"]
source: "https://kubernetes.io/docs/concepts/workloads/controllers/deployment/"
created: 2026-10-01
score: 0.903
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Kubernetes reports a stalled Deployment rollout but never rolls it back by itself

## Core idea
In Kubernetes (v1.37 docs), a Deployment rollout can get stuck, for example on readiness probe
failures, image pull errors or insufficient quota. `.spec.progressDeadlineSeconds`, which defaults
to 600, sets how long the Deployment controller waits before it reports the stall: it then sets
the Deployment's `Progressing` condition to status `False` with reason `ProgressDeadlineExceeded`.
The docs state that Kubernetes takes no action on a stalled Deployment other than to report this
status condition, and that higher-level orchestrators can act on it, for example by rolling the
Deployment back. `kubectl rollout status` returns a non-zero exit code once the deadline is
exceeded. Meanwhile the Deployment controller stops scaling up the new ReplicaSet, within the
`maxUnavailable` bound, so the old Pods keep serving.

## Why choose / why not
- Rely on the deadline when: the pipeline runs `kubectl rollout status` or watches the condition
  and then decides to roll back; it turns a hang into a failure signal.
- Don't expect self-healing: without a pipeline step, `helm upgrade --rollback-on-failure` or a
  progressive-delivery controller, a stalled rollout stays half done, with two ReplicaSets.
- Raise `progressDeadlineSeconds` when: the application legitimately starts slowly; the value
  must be greater than `minReadySeconds`.

## Interview angle
- Probed as "does Kubernetes roll back a failed deployment?"; the answer is no, and the follow-up
  is who does.
- Common wrong answer: "Kubernetes detects the failure and rolls back automatically."
- Strong answer: name the `ProgressDeadlineExceeded` condition and the exit code of
  `kubectl rollout status`, then say which component in your pipeline reacts to them.

## Related
- [[kubectl rollout undo rolls back only the Deployment's Pod template]]: that note covers what a
  manual rollback restores; this one covers why someone has to trigger it.
- [[A Deployment rolling update is bounded by maxSurge and maxUnavailable]]: the `maxUnavailable`
  bound is what keeps the old Pods serving while the rollout is stuck.
