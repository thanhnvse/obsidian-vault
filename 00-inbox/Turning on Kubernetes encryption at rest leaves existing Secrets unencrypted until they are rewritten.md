---
tags: [security, kubernetes, secrets, interview]
status: draft
author: claude
up: ["[[Security MOC]]"]
source: "https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/"
created: 2026-10-01
score: 0.827
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Turning on Kubernetes encryption at rest leaves existing Secrets unencrypted until they are rewritten

## Core idea
The Kubernetes API server encrypts a Secret when it writes it to etcd, using the first provider
listed in its `EncryptionConfiguration`. Enabling a provider therefore does nothing for the
Secrets already stored: they stay as they were written, which under the default `identity`
provider means plaintext, until something writes them again. The Kubernetes documentation
(v1.37) says it is often not enough to make sure new objects get encrypted, and rewrites every
Secret with `kubectl get secrets --all-namespaces -o json | kubectl replace -f -`, which reads
each Secret and updates it with the same data. It also warns that seeing a non-identity provider
first in the configuration does not tell you whether an earlier migration succeeded.

## Why choose / why not
- Run the rewrite when: you have just enabled encryption, or changed the first provider, on a
  cluster that already holds Secrets; the documentation notes the command is safe to run more
  than once.
- Split the rewrite by namespace when: the cluster is large, because one command that reads and
  replaces every Secret puts the whole load on the API server at once.
- Don't treat the configuration file as proof: verify by reading a Secret's stored value
  directly from etcd, as the documentation's verification step does.

## Interview angle
- Probed as "you enabled encryption at rest last week; is every Secret in etcd encrypted now?"
- Common wrong answer: "Yes, the API server encrypts everything once the provider is configured."
- Strong answer: only writes go through the provider, so existing Secrets must be rewritten; name
  the replace command and how to verify one Secret in etcd.

## Related
- [[A Kubernetes Secret is only base64-encoded and is stored unencrypted in etcd by default]]:
  that note establishes that encryption at rest is opt-in; this note covers the step people miss
  after opting in.
