---
tags: [grpc, kubernetes, load-balancing, microservices, interview]
status: draft
author: claude
up: ["[[Microservices and messaging MOC]]", "[[Kubernetes MOC]]"]
source: "https://github.com/grpc/grpc/blob/master/doc/load-balancing.md"
created: 2026-10-07
review: unjudged
---
# A gRPC client with one channel behind a per-connection balancer such as a ClusterIP Service sends every call to one pod until it reconnects

## Core idea
A Kubernetes Service in iptables mode picks a backend when a client connects, randomly or by session
affinity, so the choice is made per connection. gRPC runs on HTTP/2, which multiplexes many calls on
one connection, and a channel typically keeps one connection to its target, so every call goes to the
pod chosen at connect time; the gRPC documentation says balancing within gRPC happens per call, not
per connection. In the write-up's test a small TCP proxy standing in for the Service gave each
accepted connection to the next of three backends: 30 calls on one channel went 30, 0, 0, while three
channels gave 10, 10, 10. With one channel and a resolver that returns three addresses, the default
`pick_first` policy leaves two backends idle (rỗi, không nhận việc) and `round_robin` gives 10, 10, 10.

## Why choose / why not
- Keep a plain ClusterIP Service when: clients are HTTP/1.1 REST with short-lived connections; many
  connections give many choices and nothing extra is needed.
- Use a headless Service plus `round_robin` in the channel when: the client is gRPC and latency
  matters most; there is no extra hop, but the client is more complex and a new pod joins only when
  the resolver answers again, a timing that differs per client.
- Use an L7 proxy or a mesh when: every client language must be balanced per request; it spreads
  HTTP/2 streams, at the price of an extra hop and higher latency.
- Use several channels only when: you want no new component; calls spread only as far as there are
  connections.
- Don't expect scaling out to help: with one channel behind a ClusterIP, new pods get no traffic
  until clients reconnect.

## Interview angle
- Probed as "you call a gRPC service through a ClusterIP and one pod is hot".
- Common wrong answers: "gRPC load-balances itself", though the default policy is `pick_first`, or
  "add more pods".
- Strong answer: the Service balances per connection and gRPC puts all calls on one HTTP/2
  connection; then the fixes with their costs, the DNS re-resolution caveat, and per-pod request
  counts as the evidence.

## Related
- [[Microservices and messaging MOC]]: the map for what happens between services; this is the
  balancing trap behind gRPC calls.
- [[Kubernetes MOC]]: the Service is the Kubernetes piece that makes this per-connection choice.
- [[A Kubernetes canary built from two Deployments splits traffic by replica count]]: it relies on
  the same Service, and its caveat that the split is per connection is why one long-lived gRPC
  connection does not spread.
- [[A StatefulSet Pod keeps its DNS name across restarts but not its IP address]]: explains the
  headless Service, where DNS returns Pod addresses and nothing balances, which the client-side fix
  builds on.
- Written up in win-interview: backend/docs/microservice-infrastructure.md, section 2.5
