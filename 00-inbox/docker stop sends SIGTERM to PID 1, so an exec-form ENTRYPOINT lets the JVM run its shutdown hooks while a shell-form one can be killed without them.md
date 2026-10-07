---
tags: [docker, jvm, shutdown, ops, interview]
status: draft
author: claude
up: ["[[Ops and cloud MOC]]"]
source: "https://docs.docker.com/reference/dockerfile/"
created: 2026-10-07
review: unjudged
---
# docker stop sends SIGTERM to PID 1, so an exec-form ENTRYPOINT lets the JVM run its shutdown hooks while a shell-form one can be killed without them

## Core idea
`docker stop` sends SIGTERM to the container's main process (PID 1) and, after a grace period
(10 s by default on Linux), SIGKILL. The JVM installs a SIGTERM handler that runs its shutdown
hooks, but `/bin/sh -c "java ..."` does not forward the signal to its child: Docker's reference
says the shell form of `ENTRYPOINT` runs as a subcommand of `/bin/sh -c`, which does not pass
signals. In a lab (`PidOneSignalIT`, Docker 27.3.1, Temurin 21.0.12.1) exec form and
`sh -c "exec java ..."` ran the hook and exited 143 (128 + SIGTERM); `sh -c "java ..."` ran no hook
and exited 137 (128 + SIGKILL), with `sh` as PID 1. In a manual run of that form the JVM was alive
as PID 7 and never heard the signal. Whether `sh -c` replaces itself with its last command is the
shell's choice, and dash did not, so write the form you mean.

## Why choose / why not
- Choose exec form (`ENTRYPOINT ["java", ...]`) when: the container must drain (finish the requests
  in flight) on stop; the JVM is PID 1, so its shutdown hooks and Spring Boot's graceful shutdown run.
- Use a wrapper script when: you must keep one; it is a shell too, so end it with `exec java ...`
  so the JVM replaces the shell.
- Add `docker run --init` when: the main process is not a JVM, or it spawns children that need
  reaping; it runs an init that forwards signals and reaps processes.
- Don't write `CMD java -jar app.jar` or `ENTRYPOINT java ...` when `sh` would stay PID 1:
  rollouts wait out the full grace period and no shutdown line is logged; `cat /proc/1/cmdline`
  inside the container showing `sh -c` confirms it.

## Interview angle
- Probed as "what is PID 1 and why does it matter for a Java service?"; the symptom that points
  to it is a rollout that always takes the full grace period.
- Common wrong answer: "`CMD java -jar app.jar` is fine."
- Strong answer: `docker stop` signals PID 1; only a process with a handler receives SIGTERM and a
  shell does not forward it; the JVM as PID 1 runs its hooks and exits 143, a shell-wrapped one
  is killed after the grace period with 137. Then tie it to graceful shutdown and rollouts.

## Related
- [[A terminating Pod can still receive requests after TERM because its endpoint removal runs concurrently]]:
  that note covers what Kubernetes does around the TERM signal (preStop, endpoint removal, grace
  period); this one covers whether the JVM ever receives it.
- [[Ops and cloud MOC]]: how a container stops decides whether a release drops requests, which is
  the "how does this ship" side of the map.
- Written up in win-interview: ops/docker-and-ci.md, section 2.5
