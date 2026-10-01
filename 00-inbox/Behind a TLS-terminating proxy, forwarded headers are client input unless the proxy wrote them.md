---
tags: [security, tls, https, spring, interview]
status: draft
author: claude
up: ["[[Security MOC]]"]
source: "https://www.rfc-editor.org/rfc/rfc7239.html#section-8.1"
created: 2026-09-30
score: 0.875
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Behind a TLS-terminating proxy, forwarded headers are client input unless the proxy wrote them

## Core idea
When a load balancer or a Kubernetes Ingress terminates TLS, the application receives plain HTTP
from the proxy; the Kubernetes Ingress documentation says traffic to the Service and its Pods is
in plaintext. The original scheme and client address then reach the application only through
headers the proxy adds: the standard `Forwarded` header of RFC 7239, or the non-standard
`X-Forwarded-For` and `X-Forwarded-Proto`. RFC 7239 warns that the `Forwarded` header cannot be
relied upon to be correct, because any node on the way, including the client, may modify it. An
application should therefore accept these headers only from its own proxies.

## Why choose / why not
- Turn on forwarded-header handling when: the application builds redirects or links, or makes
  security decisions, from the scheme or the client IP; in Spring Boot 4.1.1 that is
  `server.forward-headers-strategy`, which defaults to `NATIVE` on supported cloud platforms such
  as Kubernetes and to `NONE` elsewhere.
- Restrict the trusted proxies when you turn it on: Tomcat trusts only proxies matching
  `server.tomcat.remoteip.internal-proxies`, and Spring Boot says not to trust all proxies in
  production.
- Don't base rate limits or IP allow-lists on `X-Forwarded-For` unless the edge proxy overwrites
  it: otherwise the client chooses the value.

## Interview angle
- Probed as "after we moved TLS to the ingress, redirects go to `http://`; why?" or "how do you
  get the client IP behind a load balancer?"
- Common wrong answer: "Just read `X-Forwarded-For`."
- Strong answer: the app now sees the proxy's plain-HTTP connection; scheme and client IP come
  from forwarded headers, which count only when a trusted proxy wrote them.

## Related
- [[HTTPS hides the request path and headers but not the IP addresses or the SNI hostname]]: that
  note is about what an observer on the network sees; this note is about what the backend sees
  once TLS has ended at the proxy.
