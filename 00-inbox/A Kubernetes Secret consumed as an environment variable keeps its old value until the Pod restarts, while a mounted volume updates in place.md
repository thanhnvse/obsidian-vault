---
tags: [security, kubernetes, secrets, interview]
status: draft
author: claude
up: ["[[Security MOC]]"]
source: "https://kubernetes.io/docs/concepts/configuration/secret/"
created: 2026-10-01
score: 0.897
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A Kubernetes Secret consumed as an environment variable keeps its old value until the Pod restarts, while a mounted volume updates in place

## Core idea
Whether an updated Secret reaches a running container depends on how the container consumes it.
Kubernetes sets environment variables when the container starts, so a value taken from a Secret
through `env` or `envFrom` stays the same until the container restarts; the Kubernetes ConfigMap
documentation (v1.37) states this rule for that mechanism: values consumed as environment
variables are not updated automatically and require a Pod restart. A Secret mounted as a volume
behaves differently: when the Secret is updated, Kubernetes updates the data in the volume using
an eventually-consistent approach, with a delay of up to the kubelet sync period plus the cache
propagation delay. The exception is a `subPath` volume mount, which does not receive automated
Secret updates.

## Why choose / why not
- Choose a volume mount when: the credential rotates and the application re-reads the file; the
  new value then arrives without a restart, but only once the application reads the file again.
- Choose an environment variable when: the value changes rarely and a rolling restart is an
  acceptable step of every rotation; every framework reads environment variables.
- Don't mount a rotating Secret with `subPath`: it never updates; mount the whole volume and point
  the application at the file inside it.

## Interview angle
- Probed as "we rotated the database password in the Secret; why does the service still use the
  old one?"
- Common wrong answer: "Kubernetes pushes Secret updates into running Pods, so it must be a caching
  bug in the application."
- Strong answer: environment variables are fixed when the container starts; volume mounts update
  eventually, but not through `subPath`, and the application must still re-read the file. Pick
  the delivery by how the secret rotates.

## Related
- [[HashiCorp Vault dynamic secrets are generated per client with a lease and can be revoked]]:
  credentials that expire only work if the application can pick up new values, the same problem
  this note describes for Secrets.
- [[A Kubernetes Secret is only base64-encoded and is stored unencrypted in etcd by default]]: that
  note is about who can read a Secret where it is stored; this note is about how its value
  reaches the container and when it changes.
