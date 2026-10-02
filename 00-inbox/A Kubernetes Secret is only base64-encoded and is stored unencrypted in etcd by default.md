---
tags: [security, kubernetes, secrets, interview]
status: draft
author: claude
source: "https://kubernetes.io/docs/concepts/configuration/secret/"
created: 2026-09-30
score: 0.856
review: "borderline"
score_reasons: ["atomic: 0.53 (borderline)", "numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 2
---
# A Kubernetes Secret is only base64-encoded and is stored unencrypted in etcd by default

## Core idea
The values in a Secret's `data` field are base64-encoded strings; the Kubernetes documentation
(v1.37) says base64 obscures them but does not provide any useful level of confidentiality. By
default the API server stores Secrets unencrypted in etcd, so anyone with API access can retrieve
or modify a Secret, and so can anyone with access to etcd. Anyone allowed to create a Pod in a
namespace can use that access to read any Secret in that namespace, including indirectly through
the right to create a Deployment. Encryption at rest is opt-in: the API server needs an
`--encryption-provider-config` file, because the default `identity` provider writes resources
without encryption.

## Why choose / why not
- Use a plain Kubernetes Secret when: encryption at rest is configured, preferably with the KMS v2
  provider (stable since v1.29) so the key-encryption key lives in an external KMS, and RBAC grants
  read access to Secrets only to the workloads that need them.
- Remember that RBAC on Secrets is not enough when: people can create Pods in the namespace; limit
  who can create workloads there, or move sensitive Secrets to their own namespace.
- Use an external secret store instead when: credentials must rotate, be audited per consumer or
  expire on their own; a Secret holds a static value until someone changes it.
- Never commit a Secret manifest to Git: base64 in a repository is plaintext for everyone who can
  clone it.

## Interview angle
- Probed as "are Kubernetes Secrets secure?" or "where does your service's database password
  live?"
- Common wrong answer: "Yes, Secrets are encrypted", mistaking base64 for encryption.
- Strong answer: base64 is an encoding; name encryption at rest with a KMS provider, least-privilege
  RBAC including the Pod-creation path, and an external store for credentials that must rotate.

## Related
- [[A signed JWT is readable by anyone who holds it]]: both notes correct the same mistake,
  treating base64 encoding as protection; a JWT payload and a Secret's `data` field can each be
  decoded by anyone who gets a copy.
- [[Security MOC]]: the map's Security section asks who can read a piece of data;
  this note answers that question for credentials stored in the cluster.
