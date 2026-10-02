---
tags: [kubernetes, probes, ops, interview]
status: draft
author: claude
up: ["[[Kubernetes MOC]]"]
source: "https://kubernetes.io/docs/concepts/configuration/liveness-readiness-startup-probes/"
created: 2026-10-01
score: 0.87
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A failed liveness probe restarts the container while a failed readiness probe only stops its traffic

## Core idea
In Kubernetes (v1.37 docs), a failing liveness probe and a failing readiness probe have opposite
effects on a container. When a liveness probe fails `failureThreshold` times in a row (default 3),
the kubelet restarts the container. When a readiness probe fails, the container keeps running, the
Pod's Ready condition becomes false, and the EndpointSlice controller removes the Pod's IP address
from the EndpointSlices of every Service that matches the Pod. Because a liveness failure means a
restart, Spring Boot's documentation (4.1.1) warns that a liveness probe should not depend on
external systems: if a shared database fails, Kubernetes might restart every application instance.

## Why choose / why not
- Put a check in liveness when: a restart can fix what it detects, such as a deadlocked process;
  keep it to the process itself.
- Put a check in readiness when: the instance cannot serve right now, for example while it is
  still warming up; it leaves rotation without being killed.
- Don't put a shared dependency in liveness: a database outage then restarts every replica at
  once. For readiness it is a judgement call, because if every replica turns unready the Service
  has no endpoints left.

## Interview angle
- Probed as "liveness vs readiness for a Spring Boot service?"; the test is whether you know what
  each failure does before you say what to check.
- Common wrong answer: "both probes call the same health endpoint, database included."
- Strong answer: restart vs removal from Service endpoints, then derive the contents: process
  health in `/actuator/health/liveness`, ability to serve in `/actuator/health/readiness`, and no
  shared dependency in liveness.

## Related
- [[A Deployment rolling update is bounded by maxSurge and maxUnavailable]]: a new Pod only counts
  as available once it is ready, so readiness paces that rollout; this note covers what each probe
  failure does to a running Pod.
