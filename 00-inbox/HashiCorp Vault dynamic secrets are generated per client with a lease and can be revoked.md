---
tags: [security, vault, secrets, interview]
status: draft
author: claude
source: "https://developer.hashicorp.com/vault/docs/concepts/lease"
created: 2026-09-30
score: 0.906
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# HashiCorp Vault dynamic secrets are generated per client with a lease and can be revoked

## Core idea
A Vault dynamic secret is generated when a client asks for it: the database secrets engine, for
example, creates database credentials from a configured role, so every service reaches the
database with its own unique credentials. The Vault documentation (v2.x) requires every dynamic
secret to have a lease with a time to live, and the consumer must renew the lease within that
time or request a replacement secret. When a lease expires, Vault revokes it automatically;
revoking a lease invalidates the secret immediately and prevents further renewals. Operators can
revoke a single lease by its `lease_id`, or a whole tree of leases by path prefix, because a lease
ID always starts with the path the secret was requested from.

## Why choose / why not
- Choose dynamic database credentials when: many service instances reach one database and each
  leak must be traceable to one SQL username and revocable alone, without rotating a password
  every other instance shares.
- Don't choose them when: the application cannot renew leases and reconnect with new credentials
  before the old ones expire; a static secret in the KV engine, which issues no leases, is the
  simpler fit, at the cost of manual rotation.
- Don't choose them when: the target system cannot create and drop accounts on demand; there is
  nothing for Vault to generate or revoke.

## Interview angle
- Probed as "a service instance was compromised; what do you revoke, and what keeps running?"
- Common wrong answer: "Rotate the shared database password", which breaks every other instance
  at the same moment.
- Strong answer: short-lived per-instance credentials with a lease; revoke the one lease, or the
  prefix, that leaked, and let the other instances keep their own credentials.

## Related
- [[A Kubernetes Secret is only base64-encoded and is stored unencrypted in etcd by default]]: a
  Kubernetes Secret holds a static value until someone changes it; this note is the alternative
  when credentials must expire and be revoked one by one.
- [[A self-contained JWT stays valid until it expires unless verifiers check revocation state]]:
  both notes weigh expiry against revocation, but a Vault lease can be revoked at once because
  Vault itself invalidates the credential it created.
