---
tags: [cloud, aws, kubernetes, security, ops, interview]
status: draft
author: claude
up: ["[[Ops and cloud MOC]]", "[[Kubernetes MOC]]"]
source: "https://docs.aws.amazon.com/eks/latest/best-practices/security.html"
created: 2026-10-01
score: 0.867
review: "borderline"
score_reasons: ["atomic: 0.55 (borderline)"]
judged_by: "jev-1.13.0"
judge_rounds: 2
---
# On Amazon EKS AWS runs the control plane, while who patches the nodes depends on the node type

## Core idea
The Amazon EKS Best Practices Guide draws the shared responsibility line for EKS through the worker
nodes. AWS always manages the EKS control plane, including the Kubernetes control plane nodes and
the etcd database, but for infrastructure security AWS takes on more responsibility as the customer
moves from self-managed worker nodes to managed node groups to Fargate. With managed node groups,
AWS keeps the EKS optimized AMI up to date with Kubernetes patch versions and security patches, but
the customer is responsible for upgrading the node groups to the latest AMI. With Fargate, AWS is
responsible for securing the underlying instance and runtime used to run the Pods.

## Why choose / why not
- Choose Fargate or managed node groups when: the team does not want to build node provisioning
  and patching itself; Fargate takes the instance and runtime off the team, while managed node
  groups still need you to roll out new AMIs.
- Choose self-managed nodes when: you need custom AMIs, kernel settings or node agents, and you
  accept owning their patches and hardening.
- Don't treat any node type as covering the workloads: the guide leaves IAM, pod security,
  runtime security and network security largely with the customer, so a CVE in your image is still
  yours.

## Interview angle
- Probed as "EKS runs Kubernetes for us, so the nodes are patched, right?"
- Common wrong answer: "yes, a managed Kubernetes service patches everything."
- Strong answer: the control plane and etcd are AWS's; node patching depends on self-managed nodes,
  managed node groups or Fargate; workloads, images and access control are always yours.

## Related
- [[The shared responsibility model leaves data and IAM with the customer even on managed services]]:
  the general model; this note applies it to managed Kubernetes, where the line runs through the
  worker nodes.
