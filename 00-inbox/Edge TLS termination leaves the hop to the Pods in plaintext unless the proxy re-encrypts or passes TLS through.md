---
tags: [security, tls, https, kubernetes, interview]
status: draft
author: claude
up: ["[[Security MOC]]"]
source: "https://gateway-api.sigs.k8s.io/guides/tls/"
created: 2026-10-01
score: 0.896
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Edge TLS termination leaves the hop to the Pods in plaintext unless the proxy re-encrypts or passes TLS through

## Core idea
Terminating TLS means the TLS session ends at that proxy: the proxy holds the certificate and the
private key, decrypts, and forwards the request. The Kubernetes Ingress documentation (v1.37) says
Ingress assumes TLS termination at the ingress point, with traffic to the Service and its Pods in
plaintext. Gateway API offers two alternatives. With `BackendTLSPolicy`, the connection is
terminated and then re-encrypted at the Gateway, so the hop to the backend is a second TLS
session, while the Gateway still sees the plaintext in between. In `Passthrough` mode the client's
TLS session is not terminated at the Gateway but passes through it encrypted, so only the backend
holds the key, and the Gateway cannot route on HTTP paths or headers because it never sees them.

## Why choose / why not
- Choose edge termination when: the network between the proxy and the Pods is trusted and
  isolated, and you want path routing, header injection or a web application firewall at the
  proxy.
- Choose terminate and re-encrypt when: internal traffic must be encrypted too, for example under a
  compliance rule; you then issue and rotate a second set of certificates for the backends.
- Choose passthrough when: the proxy must not see plaintext, or the backend needs the client's
  certificate for mutual TLS; you give up HTTP-level routing at the proxy, and every backend
  manages its own certificate.

## Interview angle
- Probed as "we have HTTPS on the ingress; is the traffic encrypted all the way to the Pod?"
- Common wrong answer: "Yes, TLS at the load balancer means the whole path is encrypted."
- Strong answer: name the three modes and what the proxy sees in each. Edge termination is
  plaintext to the Pod, re-encryption adds a second TLS session, and passthrough leaves the proxy
  blind to HTTP.

## Related
- [[Behind a TLS-terminating proxy, forwarded headers are client input unless the proxy wrote them]]:
  that note covers what the application must do once TLS ends at the proxy; this note covers the
  choice of where TLS ends.
- [[HTTPS hides the request path and headers but not the IP addresses or the SNI hostname]]: a
  passthrough proxy is in the position of that note's on-path observer, seeing the SNI host name
  but not the path, so it can route only by host name.
