---
tags: [kubernetes, statefulset, networking, ops, interview]
status: draft
author: claude
up: ["[[Kubernetes MOC]]"]
source: "https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set/"
created: 2026-10-01
score: 0.887
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A StatefulSet Pod keeps its DNS name across restarts but not its IP address

## Core idea
In Kubernetes (v1.37 docs), the stable network identity of a StatefulSet Pod is its hostname and
DNS name, not its IP address. A StatefulSet needs a headless Service, which you create yourself,
to be responsible for the network identity of its Pods. A headless Service sets
`clusterIP: None`: no cluster IP is allocated, kube-proxy does not handle it, and the platform
does no load balancing or proxying for it; instead, the cluster DNS reports the IP addresses of
the individual Pods. Each Pod
gets a DNS name of the form `$(podname).$(governing service domain)`, such as
`web-0.nginx.default.svc.cluster.local`. When the StatefulSet's Pods are deleted and recreated,
their ordinals, hostnames, SRV records and A record names stay the same, but the IP addresses
associated with the Pods may change.

## Why choose / why not
- Address a member by its per-Pod DNS name when: clients or peers must reach one specific member,
  such as a database primary or a Kafka broker; the name follows the ordinal across restarts.
- Don't pin a StatefulSet Pod's IP in a client or a config file: it can change after any
  restart, so the client should resolve the hostname again when it reconnects.
- Add an ordinary ClusterIP Service over the same Pods when: clients want any healthy member and
  load balancing; the headless Service only answers DNS and balances nothing.

## Interview angle
- Probed as "what is a headless Service for?" or "do StatefulSet Pods keep their IP?"
- Common wrong answer: "StatefulSet Pods keep their IP address." The name and the DNS record are
  stable; the IP is not.
- Strong answer: `clusterIP: None`, no proxying, DNS returns Pod addresses; the stable part is the
  per-ordinal name, so clients connect by hostname and re-resolve after a restart.

## Related
- [[A StatefulSet gives each pod a stable identity and its own PersistentVolumeClaim]]: that note
  lists the three parts of the identity; this one covers the network part, what the headless
  Service resolves, and that the IP address is not part of it.
