---
tags: [java, jvm, gc, docker, interview]
status: draft
author: claude
up: ["[[Java core MOC]]", "[[Ops and cloud MOC]]"]
source: "https://docs.oracle.com/en/java/javase/21/gctuning/ergonomics1.html"
created: 2026-10-07
review: unjudged
---
# By default in JDK 21, a container with one CPU or under 1792 MB gets the Serial collector instead of G1

## Core idea
JDK 21 reads the container's cgroup limits (`UseContainerSupport` is on by default), so the
processor count and the memory it sees are the limits, not the host's. Its ergonomics treat a
machine as server-class only with two or more processors and at least 1792 MB; a container below
that gets the Serial collector, which pauses all threads, instead of G1. A lab (`ContainerAwareJvmIT`,
Docker 27.3.1 on a 12-CPU VM, Temurin 21.0.12.1) printed `UseG1GC` for 2 GiB and 2 CPUs, but
`UseSerialGC` for 2 GiB and 1 CPU and for 1 GiB and 2 CPUs; an explicit `-XX:+UseG1GC` at 1 GiB and
1 CPU gave G1. A fractional CPU limit is rounded up (0.5 gives 1, 1.5 gives 2). The test samples
both sides of the rule, not its exact boundary.

## Why choose / why not
- Read the flags (`-XX:+PrintFlagsFinal | grep UseSerialGC`, or the GC log) when: you set or change
  a container's limits; the collector can change with the limit alone and nobody chose it.
- Ask for G1 or ZGC explicitly when: a small container must not run Serial; an explicit collector
  flag overrides the ergonomic choice.
- Give the container a limit of two CPUs and at least 1792 MB when: you want the JVM to treat it as
  server-class and keep the default G1; the JVM reads the limit, not the host.
- Don't assume the laptop's collector is production's: the same jar in a 1 CPU, 1 GiB container
  runs a different collector from the same jar on a developer machine.

## Interview angle
- Probed as "how does the JVM know the container's memory and CPU?"; a strong answer goes on to the
  collector it picks.
- Common wrong answer: "the JVM does not know about containers".
- Strong answer: JDK 21 reads the cgroup limits by default; below two CPUs or 1792 MB it picks
  Serial; read the flags in the pod instead of assuming, and name the collector explicitly if it
  matters.

## Related
- [[ZGC trades some throughput for sub-millisecond pauses, so G1 stays the default collector]]:
  that note chooses between G1 and ZGC on a server-class machine; this one covers the container
  that never gets G1 because it is too small.
- [[Java core MOC]]: its garbage-collection section is where the collector-choice notes live,
  and this default sits underneath all of them.
- [[Ops and cloud MOC]]: container sizing decides which collector runs, so it belongs with how a
  service is shipped and sized.
- Written up in win-interview: ops/docker-and-ci.md, section 2.4
