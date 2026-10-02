---
tags: [security, secrets, docker, interview]
status: draft
author: claude
up: ["[[Security MOC]]"]
source: "https://docs.docker.com/build/building/secrets/"
created: 2026-10-01
score: 0.917
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 2
---
# Docker build arguments and ENV values persist in the image, so a build needs secret mounts for credentials

## Core idea
A credential that a Dockerfile receives through `ARG` or `ENV` can be read back from the image it
builds. Docker's documentation says build arguments and environment variables are inappropriate
for passing secrets to a build, because they persist in the final image; the Dockerfile reference
adds that build arguments are visible in the `docker history` command, and that values set with
`ENV` persist when a container is run from the resulting image. A credential needed only during
the build, for example to download dependencies from a private repository, is therefore exposed to
everyone who can pull the image. A secret mount avoids this: it takes the secret from the build
client and makes it temporarily available inside the build container for the duration of one
build instruction, by default as a file at `/run/secrets/<id>`.

## Why choose / why not
- Choose a secret mount, `RUN --mount=type=secret`, when: one build step needs a credential, such
  as access to a private package repository; only that `RUN` instruction can read it.
- Don't use `ARG` on the grounds that it is not an `ENV` in the final image: Docker states that
  build arguments stay visible in `docker history`.
- Don't put a build credential in `ENV` for convenience: it persists into every container started
  from the image.

## Interview angle
- Probed as "how does your Dockerfile get the credentials for the private Maven repository?"
- Common wrong answer: "We pass them as a build argument, so they are not in the image."
- Strong answer: build arguments and `ENV` values persist in the image and its history; use a
  BuildKit secret mount, which gives the credential to one `RUN` instruction only.

## Related
- [[A secret committed to Git must be revoked or rotated first, because removing it from history does not undo the leak]]:
  an image pushed to a registry is copied like a pushed commit, so a credential baked into it
  needs the same revoke-first response.
- [[A Pod can log in to Vault with its service-account token, so it needs no static secret to reach Vault]]:
  that note covers how a running container gets its secrets without one baked in; this note
  covers the build, which runs before any platform identity exists.
