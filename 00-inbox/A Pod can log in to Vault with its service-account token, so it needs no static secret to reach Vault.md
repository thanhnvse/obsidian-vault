---
tags: [security, vault, kubernetes, secrets, interview]
status: draft
author: claude
up: ["[[Security MOC]]"]
source: "https://developer.hashicorp.com/vault/docs/auth/kubernetes"
created: 2026-10-01
score: 0.869
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A Pod can log in to Vault with its service-account token, so it needs no static secret to reach Vault

## Core idea
A service needs a Vault token before Vault gives it any secret, and if that token came from one
more static secret in the service's configuration, the problem would only move. Vault's
Kubernetes auth method lets a workload authenticate with its Kubernetes service account token,
and Vault checks that token with the Kubernetes TokenReview API. A Vault role binds service
account names and namespaces to policies and a token TTL, for example
`bound_service_account_names=myapp`, `bound_service_account_namespaces=default` and `ttl=1h`,
so only Pods running as that account in that namespace receive that policy.

## Why choose / why not
- Choose Kubernetes auth when: the workload runs in a cluster whose API Vault can reach for
  TokenReview; the identity then comes from the platform and needs no secret in the image or the
  configuration.
- Bind each Vault role narrowly: one service account in one namespace per role, because every Pod
  that runs as a bound account gets the role's policies.
- Don't hand the service-account token to anything else: the Vault documentation warns that the
  token allows API calls on behalf of the Pod, so sharing it can grant unintended access.

## Interview angle
- Probed as "how does your service authenticate to Vault without a password in its config?"
- Common wrong answer: "We store the Vault token in a Kubernetes Secret."
- Strong answer: use the platform identity, the service-account token, through Vault's Kubernetes
  auth method; the role maps that identity to a narrow policy and a short TTL.

## Related
- [[HashiCorp Vault dynamic secrets are generated per client with a lease and can be revoked]]:
  that note covers what Vault hands out; this one covers how the workload proves who it is before
  Vault hands out anything.
- [[A Kubernetes Secret is only base64-encoded and is stored unencrypted in etcd by default]]:
  keeping a Vault token in a Secret would bring back every weakness that note lists.
