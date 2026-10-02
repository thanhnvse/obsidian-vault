---
tags: [deployment-strategy, ops, interview]
status: draft
author: claude
source: "https://martinfowler.com/bliki/CanaryRelease.html"
created: 2026-09-30
score: 0.853
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Blue-green switches all traffic at once while a canary shifts a subset of users first

## Core idea
Blue-green and canary differ in how much live traffic moves to the new version at once.
Blue-green keeps two production environments as identical as possible, one of them live; the
new version goes to the idle one, then the router is switched so that all incoming requests go
to it, and a rollback switches the router back for everyone. A canary release deploys the new
version where no users reach it, then routes a few selected users to it and more users as
confidence grows; a rollback reroutes only those users to the old version. Because a canary
moves traffic gradually, several versions serve users at once for longer and must all be
managed.

## Why choose / why not
- Choose blue-green when: you need an instant, all-or-nothing switch and an equally fast switch
  back, and you can pay for a second full production environment during the release.
- Choose a canary when: you want real production traffic to prove the release on a small blast
  radius first; it costs traffic routing, per-version metrics to compare, and a longer period
  with two versions live.
- Choose neither when: the change is low risk and a plain rolling update is enough, or when the
  old and new versions cannot share the database; fix that compatibility first.

## Interview angle
- Probed as "how would you release a risky change to the payment flow?"; the answer should
  compare the speed of rollback with the size of the blast radius.
- Common wrong answer: "a rolling update is a canary." A rolling update replaces Pods at a set
  rate until it finishes or stalls; it does not compare the new version's metrics first.
- Strong answer: tie the choice to the rollback: blue-green undoes the release for every user
  with one switch, while a canary limits the damage to the users already routed to it.

## Related
- [[A Deployment rolling update is bounded by maxSurge and maxUnavailable]]: the rolling update
  is the default these two strategies are measured against, and it also runs two versions at
  once while it progresses.
- [[Ops and cloud MOC]]: this answers the Ops and cloud question of how a risky release
  ships and how it comes back.
