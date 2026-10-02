---
tags: [kubernetes, helm, crd, ops, interview]
status: draft
author: claude
up: ["[[Kubernetes MOC]]"]
source: "https://helm.sh/docs/chart_best_practices/custom_resource_definitions/"
created: 2026-10-01
score: 0.867
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Helm installs the CRDs in a chart's crds directory but never upgrades or deletes them

## Core idea
In the Helm documentation (Helm 4.3.0 site), a chart can ship Custom Resource Definitions in a
special `crds/` directory; those files are not templated, and Helm installs them by default on
`helm install`. The CRD best-practices page states that a CRD that already exists is skipped with a
warning, and that Helm does not support upgrading or deleting CRDs, an explicit decision made
because of the danger of unintentional data loss. The Charts page, which says it is not yet
updated for Helm 4, gives the rules: because CRDs are installed globally, unlike most Kubernetes
objects, CRDs are never installed on an upgrade or a rollback, only on an install, and CRDs are
never deleted, because deleting a CRD automatically deletes all of the CRD's contents across all
namespaces in the cluster.

## Why choose / why not
- Use `crds/` when: the chart introduces a CRD that is installed once and changes rarely, such as
  the first install of an operator.
- Don't rely on it for CRD changes: a new CRD version in `crds/` is ignored by `helm upgrade`, so
  ship CRD upgrades as a separate, deliberate step, or from a separate chart that holds only the
  CRDs.
- Don't expect `helm uninstall` to remove CRDs: deleting one removes every custom resource of that
  type in every namespace, which is exactly why Helm leaves it.

## Interview angle
- Probed as "you upgraded the operator's chart; why are the new CRD fields missing?"
- Common wrong answer: "Helm upgrades my CRDs."
- Strong answer: CRDs in `crds/` are created on install only, never upgraded, rolled back or
  deleted, because a CRD is cluster-wide and deleting it deletes its data; then say how CRD
  upgrades are shipped instead.

## Related
- [[Every Helm install, upgrade or rollback creates a new release revision]]: a revision returns the
  chart's objects together, and this note names one kind of object that an upgrade or a rollback
  leaves untouched.
