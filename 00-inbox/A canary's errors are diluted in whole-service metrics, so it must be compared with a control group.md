---
tags: [deployment-strategy, observability, ops, interview]
status: draft
author: claude
up: ["[[Ops and cloud MOC]]"]
source: "https://sre.google/workbook/canarying-releases/"
created: 2026-10-01
score: 0.88
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A canary's errors are diluted in whole-service metrics, so it must be compared with a control group

## Core idea
The Google SRE Workbook defines canarying as a partial and time-limited deployment of a change in a
service and its evaluation, which decides whether the rollout proceeds. The part of the service that
receives the change is the canary, and the remainder of the service is the control; the canary is
usually a much smaller subset of production than the control. Because the canary serves a small share of traffic, its effect on
whole-service metrics is diluted: with a canary population of 5% and a release that fails 20% of
requests, the service serves 20% errors for 5% of traffic, which is a 1% overall error rate. The
Workbook therefore says that monitoring which reasons well about the entire service is not
sufficient to analyze a canary: metrics must be broken down by the population serving the
request, canary versus control, and compared.

## Why choose / why not
- Choose a canary when: the change is risky, real production traffic is the best test, and your
  metrics can be split by version; the damage is limited to the canary's share.
- Don't run a canary without per-version metrics: whole-service dashboards hide the canary's
  errors, so a broken release passes and the canary becomes a slow rollout.
- Decide the evaluation before the release: which signals, such as failed requests and crashes,
  for how long, and what result rolls the canary back.

## Interview angle
- Probed as "how do you decide that a canary is good enough to promote?"
- Common wrong answer: "if the service's error-rate dashboard stays green, promote it."
- Strong answer: compare canary with control, do the dilution arithmetic out loud, name the
  signals and the time window, and state the rollback rule in advance.

## Related
- [[A Kubernetes canary built from two Deployments splits traffic by replica count]]: that note
  builds the traffic split; this one covers how to judge what the canary shows, which needs metrics
  split by the `track` label used there.
- [[Blue-green switches all traffic at once while a canary shifts a subset of users first]]: that
  note compares the two strategies; this one covers the evaluation step that makes a canary more
  than a slow rollout.
