---
tags: [kubernetes, helm, configuration, ops, interview]
status: draft
author: claude
up: ["[[Kubernetes MOC]]"]
source: "https://helm.sh/docs/howto/charts_tips_and_tricks/#automatically-roll-deployments"
created: 2026-10-01
score: 0.883
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A Helm upgrade that changes only a ConfigMap does not restart the Pods that read it

## Core idea
In Kubernetes (v1.37 docs), a Deployment's rollout is triggered if and only if its Pod template,
`.spec.template`, changes. A `helm upgrade` that changes only the content of a ConfigMap or a
Secret updates that object but leaves the Deployment's Pod template as it was, so no rollout
starts. The Helm documentation (Helm 4.3.0 site) states that when the Deployment spec itself did
not change, the application keeps running with the old configuration, which leaves an
inconsistent deployment. Its fix is an annotation on the Pod template that holds a checksum of the
rendered ConfigMap, for example
`checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}`.
A change to the ConfigMap then changes the annotation, which changes the Pod template and rolls
the Deployment.

## Why choose / why not
- Add the checksum annotation when: the application reads its configuration only at startup, such
  as a Spring Boot service that binds properties once; each config change then rolls the Pods
  within the normal rolling-update bounds.
- Don't add it when: the application watches its mounted configuration files and reloads them by
  itself; the checksum would restart Pods that did not need it.
- Alternative: put a content hash in the ConfigMap's name, so a change creates a new ConfigMap and
  the Pod template's reference to it changes.

## Interview angle
- Probed as "why didn't my ConfigMap change take effect after `helm upgrade`?"
- Common wrong answer: "Helm restarts the application whenever the chart changes."
- Strong answer: a rollout happens only when the Pod template changes; Helm updated the ConfigMap
  object and nothing else; add a `checksum/config` annotation so the template changes with it.

## Related
- [[kubectl rollout undo rolls back only the Deployment's Pod template]]: the same fact seen from
  the rollback side; a ConfigMap's content is not part of a Deployment revision, so it neither
  starts a rollout nor comes back with a rollback.
- [[Every Helm install, upgrade or rollback creates a new release revision]]: the upgrade is still
  recorded as a new revision although no Pod restarted, which is how the stale configuration goes
  unnoticed.
- [[A Kubernetes Secret consumed as an environment variable keeps its old value until the Pod restarts, while a mounted volume updates in place]]:
  that note covers how a changed value reaches a running container; this one covers why a Helm
  upgrade does not restart the container to pick it up.
