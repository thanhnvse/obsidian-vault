---
tags: [kubernetes, deployment, shutdown, ops, interview]
status: draft
author: claude
up: ["[[Kubernetes MOC]]"]
source: "https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-termination-flow"
created: 2026-10-01
score: 0.837
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A terminating Pod can still receive requests after TERM because its endpoint removal runs concurrently

## Core idea
In Kubernetes (v1.37 docs), when a Pod is deleted, for example by a rolling update, the kubelet
first runs any `preStop` hook defined for a container and then sends the TERM signal to process 1
inside each container. At the same time as the kubelet starts this graceful shutdown, the control
plane evaluates whether to remove the shutting-down Pod from EndpointSlice objects, and endpoints
of terminating Pods are marked as not ready, so load balancers stop sending them regular traffic.
Because the two steps run concurrently instead of one after the other, the Spring Boot 4.1.1
documentation notes that there is a window during which traffic can still be routed to a Pod that
has already begun shutting down. The default grace period is 30 seconds; when it expires, the
container runtime sends SIGKILL to any process still running in the Pod.

## Why choose / why not
- Add a short `preStop` sleep when: the service sits behind a Service or a load balancer and must
  not drop requests during rollouts; the process keeps serving while endpoint removal propagates.
- Enable graceful shutdown in the application when: requests in flight at TERM must finish; Spring
  Boot lets them complete within `spring.lifecycle.timeout-per-shutdown-phase` while refusing new
  ones.
- Don't let the two exceed the grace period: the `preStop` time counts toward
  `terminationGracePeriodSeconds`, so raise the grace period instead of letting KILL cut requests
  off.

## Interview angle
- Probed as "why do we see 502s during rollouts although every new Pod passes its readiness
  probe?"
- Common wrong answer: "Kubernetes removes the Pod from the Service before it sends SIGTERM."
- Strong answer: endpoint removal and TERM happen concurrently, so add a `preStop` delay, enable
  graceful shutdown in the application, and keep both inside the grace period.

## Related
- [[A Deployment rolling update is bounded by maxSurge and maxUnavailable]]: that note covers the
  start of each Pod swap, where readiness sets the pace; this one covers its end, how an old Pod
  leaves without dropping requests.
- [[A failed liveness probe restarts the container while a failed readiness probe only stops its traffic]]:
  both rely on removing the Pod from EndpointSlices to stop traffic; here termination triggers the
  removal instead of a failed probe.
