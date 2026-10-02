---
tags: [kubernetes, deployment, ops, interview]
status: draft
author: claude
source: "https://kubernetes.io/docs/concepts/workloads/controllers/deployment/"
created: 2026-09-30
score: 0.847
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A Deployment treats its pods as interchangeable replicas of a stateless workload

## Core idea
In Kubernetes (v1.37 docs), a Deployment provides declarative updates for Pods and ReplicaSets,
and it runs a workload that usually does not maintain state. The Deployment does not manage its
Pods one by one: a change to the Pod template creates a new ReplicaSet named
`[DEPLOYMENT-NAME]-[HASH]`, which the Deployment scales up while it scales the old ReplicaSet
down. Every replica is stamped from the same Pod template, so no replica has a stable name or
storage of its own, and a PersistentVolumeClaim named in the template is one claim shared by all
replicas. The Kubernetes single-instance stateful tutorial therefore runs one replica with
`strategy: type: Recreate`, because its PersistentVolume can be mounted by only one Pod.

## Why choose / why not
- Choose when: any replica can serve any request because the state lives outside the Pod, in a
  database, a cache or an object store; this is the usual Spring Boot or Quarkus REST service.
- Don't choose when: each replica needs its own disk or a name that peers address; use a
  StatefulSet, which gives each Pod its own claim and DNS name.
- Tolerate a single stateful replica on a Deployment only with `Recreate`, and accept the
  downtime on every deploy, since a rolling update would need two Pods on one volume.

## Interview angle
- Probed as "why is the REST service a Deployment but Kafka a StatefulSet?"; the test is whether
  a replica has an identity that must survive a restart.
- Common wrong answer: "a Deployment cannot use persistent volumes." It can; it just cannot give
  each replica its own volume.
- Strong answer: say the replicas are disposable and interchangeable, then point out that the
  ReplicaSet created per template change is what makes a rolling update and a rollback possible.

## Related
- [[A StatefulSet gives each pod a stable identity and its own PersistentVolumeClaim]]: the
  contrasting controller; that note covers per-Pod identity, this one covers what a Deployment
  gives up by having none.
- [[Kubernetes MOC]]: the Ops and cloud cluster starts from this default choice for a
  stateless Java service.
