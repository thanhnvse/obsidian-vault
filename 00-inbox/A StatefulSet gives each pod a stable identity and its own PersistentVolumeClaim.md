---
tags: [kubernetes, statefulset, ops, interview]
status: draft
author: claude
source: "https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/"
created: 2026-09-30
score: 0.873
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A StatefulSet gives each pod a stable identity and its own PersistentVolumeClaim

## Core idea
In Kubernetes (v1.37 docs), a StatefulSet runs Pods from one identical spec, but each Pod gets a sticky identity made of an
ordinal, a stable network identity and stable storage, and that identity stays with the Pod
whichever node it is rescheduled on. Pods are named `$(statefulset name)-$(ordinal)`, and a
headless Service, which you must create yourself, gives each Pod its own DNS name. For each entry
in `volumeClaimTemplates`, each Pod receives its own PersistentVolumeClaim, and by default the
volumes are not deleted when the Pods or the StatefulSet are deleted or scaled down. With the default
`OrderedReady` policy, Pods are created in order from 0 to N-1 and terminated in reverse order.

## Why choose / why not
- Choose when: each replica owns its data or is addressed by name, such as a database with a
  primary and replicas, a Kafka broker, or a consensus member that peers reach by stable DNS.
- Don't choose when: the service is a stateless REST API; the ordered, one-Pod-at-a-time rollout
  only slows the deploy, and a Deployment with interchangeable replicas fits better.
- Budget for: volumes left behind after a scale-down. They are kept for data safety, so remove
  them yourself or set the PVC retention policy (`whenDeleted`, `whenScaled`; default `Retain`).

## Interview angle
- Probed as "why not run Postgres as a Deployment with a PVC?"; every replica of a Deployment
  uses the same claim from its Pod template, while `volumeClaimTemplates` give one claim per Pod.
- Common wrong answer: "a StatefulSet replicates the data." Kubernetes gives identity and
  storage; replication between members is still the database's job.
- Strong answer: name the three parts of the identity, then the costs: ordered rollout, a
  headless Service to own, and volumes that outlive the Pods.

## Related
- [[Kubernetes MOC]]: this note answers the Ops and cloud question of how a service
  ships when its replicas hold state, so the map lists it under that cluster.
