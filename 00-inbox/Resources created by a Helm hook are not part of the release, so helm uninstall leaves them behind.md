---
tags: [kubernetes, helm, hooks, ops, interview]
status: draft
author: claude
up: ["[[Kubernetes MOC]]"]
source: "https://helm.sh/docs/topics/charts_hooks/"
created: 2026-10-01
score: 0.899
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Resources created by a Helm hook are not part of the release, so helm uninstall leaves them behind

## Core idea
In the Helm documentation (Helm 4.3.0 site), a hook is an ordinary chart template with a
`helm.sh/hook` annotation, such as `pre-upgrade`, and Helm runs it at that point of the release
lifecycle. The resources a hook creates are currently not tracked or managed as part of the
release: once Helm verifies that the hook has reached its ready state, it leaves the hook resource
alone. So `helm uninstall` does not remove a hook's Job or the objects it created. Hook resources
are cleaned up through the `helm.sh/hook-delete-policy` annotation instead, whose values are
`before-hook-creation`, `hook-succeeded` and `hook-failed`; when no policy is set,
`before-hook-creation` applies, which deletes the previous hook resource before a new hook is
launched.

## Why choose / why not
- Set `hook-succeeded`, or a TTL on the Job, when: finished hook Jobs and their Pods should not
  stay in the namespace until the next upgrade.
- Use a `pre-upgrade` Job when: a step must finish before the new manifests are applied, such as a
  backup; but its effects, like a migrated schema, are outside the release and survive a
  `helm rollback`.
- Don't make a resource a hook when: it should live and die with the release, such as a ConfigMap
  the application reads; a plain template is upgraded, rolled back and uninstalled with the
  release.

## Interview angle
- Probed as "we uninstalled the release, but the migration Job is still there; why?"
- Common wrong answer: "`helm uninstall` removes everything the chart created."
- Strong answer: hook resources are not managed as part of the release; clean them up with a
  `hook-delete-policy` or a Job TTL; and what the hook did to the database is outside every
  revision.

## Related
- [[A Helm 4 upgrade without --wait can be recorded as deployed before its Pods are ready]]: that
  note covers how Helm waits for a hook Job and fails the release when it fails; this one covers
  what happens to the hook's resources afterwards.
- [[Every Helm install, upgrade or rollback creates a new release revision]]: a revision holds the
  release's own objects, and hook resources sit outside it.
