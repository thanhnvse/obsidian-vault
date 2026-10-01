---
tags: [moc, ops, cloud, interview]
type: moc
status: draft
author: claude
up: ["[[Java backend interview MOC]]"]
created: 2026-09-30
---
# Ops and cloud MOC

The question behind this map: *how does this ship, and how does it come back when it breaks?*

## Kubernetes
- [[Kubernetes MOC]]: workloads, Helm and rolling releases, and what a rollback undoes

## Release strategies
- [[Blue-green switches all traffic at once while a canary shifts a subset of users first]]: comparing the two (parked: a comparison reads as two ideas)

## Rollback
- [[Expand and contract schema changes keep the previous version runnable after a rollback]]: making rollback safe for the database

## Cloud
- [[The shared responsibility model leaves data and IAM with the customer even on managed services]]: what stays yours on a managed service
