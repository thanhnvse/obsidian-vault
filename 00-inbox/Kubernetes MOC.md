---
tags: [moc, kubernetes, ops, interview]
type: moc
status: draft
author: claude
up: ["[[Ops and cloud MOC]]"]
created: 2026-09-30
---
# Kubernetes MOC

The question behind this map: *what does Kubernetes keep running, and what does a release or a rollback change?*

## Workloads
- [[A Deployment treats its pods as interchangeable replicas of a stateless workload]]: the default workload
- [[A StatefulSet gives each pod a stable identity and its own PersistentVolumeClaim]]: when state needs identity

## Releasing
- [[Every Helm install, upgrade or rollback creates a new release revision]]: Helm's release history
- [[A Deployment rolling update is bounded by maxSurge and maxUnavailable]]: the default strategy
- [[A Kubernetes canary built from two Deployments splits traffic by replica count]]: canary without a service mesh

## Rollback
- [[kubectl rollout undo rolls back only the Deployment's Pod template]]: what a rollback does not undo
