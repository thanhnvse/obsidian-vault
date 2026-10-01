---
tags: [moc, security, interview]
type: moc
status: draft
author: claude
up: ["[[Java backend interview MOC]]"]
created: 2026-09-30
---
# Security MOC

The question behind this map: *who can read, forge or replay this?*

## JWT
- [[A signed JWT is readable by anyone who holds it]]: signed is not encrypted
- [[A JWT validator must pin the accepted algorithms instead of trusting the alg header]]: the classic validation hole
- [[A JWT with a valid signature must still be rejected when exp, iss or aud fail validation]]: a signature is not enough
- [[A self-contained JWT stays valid until it expires unless verifiers check revocation state]]: the revocation trade-off

## Transport and secrets
- [[A TLS 1.3 full handshake authenticates the server and agrees keys in one round trip]]: the HTTPS flow
- [[HTTPS hides the request path and headers but not the IP addresses or the SNI hostname]]: what an observer still sees
- [[A Kubernetes Secret is only base64-encoded and is stored unencrypted in etcd by default]]: why a Secret is not a vault
- [[HashiCorp Vault dynamic secrets are generated per client with a lease and can be revoked]]: what a vault adds
