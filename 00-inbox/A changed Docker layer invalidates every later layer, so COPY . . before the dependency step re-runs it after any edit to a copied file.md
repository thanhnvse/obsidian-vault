---
tags: [docker, build, maven, ops, interview]
status: draft
author: claude
up: ["[[Ops and cloud MOC]]"]
source: "https://docs.docker.com/build/cache/"
created: 2026-10-07
review: unjudged
---
# A changed Docker layer invalidates every later layer, so COPY . . before the dependency step re-runs it after any edit to a copied file

## Core idea
An image is a stack of read-only layers, one per filesystem-changing instruction, and the build
cache is keyed by the instruction and its inputs; for a `COPY` the input is the content of the
copied files. Docker's documentation says that when a layer changes, the layers after it are
affected too. So with `COPY . .` ahead of the dependency download, editing one source file changes
the `COPY` layer and the download step runs again. Copying only `pom.xml` first, running the
dependency step, then copying the sources leaves that step cached after a source edit, while a POM
edit still re-runs it, as it should. In a lab (`DockerLayerCacheIT`) the dependency step was
`RUN date +%s%N > /resolved`, so a cache hit and a re-run were told apart by value, not by time.

## Why choose / why not
- Copy the POMs first, then the dependency step, then the sources when: dependencies change far
  less often than code.
- Use a BuildKit cache mount (`RUN --mount=type=cache,target=/root/.m2`) when: `mvn
  dependency:go-offline` cannot be trusted. On a multi-module reactor it took 330 s here and the
  following offline build failed on a test-scope artefact it had not fetched. The step still runs
  on every edit, but `~/.m2` is already warm.
- Add a `.dockerignore` when: the context holds files that must not reach the image or the cache
  key, such as `.env`, `*.pem` and `*.key`; ignored files are not in the image and do not change the
  `COPY` layer.
- Don't count on a cache mount on a fresh CI runner: it lives in the builder's storage, and the
  `gha`, `registry` or `local` cache backends carry a cache between runs.

## Interview angle
- Probed as "why does `COPY . .` before `mvn package` make builds slow?"
- Strong answer: a changed layer invalidates all later layers, so the dependency download reruns
  on any source edit; copy the POMs first or use a cache mount, add a `.dockerignore`, and report
  what you measured, including the `go-offline` failure.

## Related
- [[Docker build arguments and ENV values persist in the image, so a build needs secret mounts for credentials]]:
  the same layer stack seen from the security side; whatever is copied or set with `ARG` or `ENV`
  lands in a layer, so `.dockerignore` keeps `.env` out of the context and secrets go through mounts.
- [[Ops and cloud MOC]]: the image build is the first step of "how does this ship", so how its
  layers are ordered belongs on that map.
- Written up in win-interview: ops/docker-and-ci.md, section 2.2
