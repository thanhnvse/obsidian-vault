---
tags: [kubernetes, helm, ops, interview]
status: draft
author: claude
up: ["[[Kubernetes MOC]]"]
source: "https://helm.sh/docs/helm/helm_upgrade/"
created: 2026-10-01
score: 0.867
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 2
---
# A Helm 4 upgrade without --wait can be recorded as deployed before its Pods are ready

## Core idea
In the Helm docs (v4.3.0), `helm upgrade` takes a `--wait` strategy of `watcher`, `hookOnly` or
`legacy`, and when the flag is omitted the default strategy is `hookOnly`. Helm does wait for hook
Jobs: it waits until a hook Job runs to completion, and a failed hook fails the release. The
Using Helm page states that Helm does not wait until all of the resources are running before it
exits. To make readiness part of the upgrade, pass `--wait` alone, which selects the `watcher`
strategy and waits until resources are ready, up to `--timeout`, whose default is 5m0s; or pass
`--rollback-on-failure`, which also defaults `--wait` to `watcher` and rolls the upgrade back to the
previous successful release when it fails.

## Why choose / why not
- Pass `--rollback-on-failure` in CI when: a release that never becomes ready must fail the
  pipeline and be reverted without a human.
- Pass `--wait` alone when: the pipeline must block until the release is ready, but a person or a
  later step decides whether to roll back.
- Don't wait in Helm when: a GitOps or progressive-delivery controller already owns readiness and
  rollback; two owners fight.
- Watch out: an automatic rollback restores the previous Kubernetes objects, not a schema that a
  `pre-upgrade` hook already migrated.

## Interview angle
- Probed as "how do you make a Helm deployment fail when the application doesn't come up?"
- Common wrong answer: "Helm waits for the Pods by default."
- Strong answer: name the default strategy, then `--wait` with `--timeout`, then
  `--rollback-on-failure` for a pipeline that must revert by itself.

## Related
- [[Every Helm install, upgrade or rollback creates a new release revision]]: that note covers the
  revision history an automatic rollback returns to; this one covers when Helm decides that an
  upgrade failed.
