---
tags: [deployment-strategy, rollback, ops, interview]
status: draft
author: claude
up: ["[[Ops and cloud MOC]]"]
source: "https://martinfowler.com/articles/feature-toggles.html"
created: 2026-10-01
score: 0.883
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A feature flag rolls back a feature without a deploy but cannot undo what the new code path wrote

## Core idea
Feature toggles, also called feature flags, let a team change system behaviour without changing
code, which makes turning a flag off a rollback of one feature that needs no deploy. Ops toggles,
including long-lived kill switches, exist for exactly this: operators disable or degrade a feature
quickly in production, so the flag must be changeable without rolling out a new release. A managed
flag service can even automate it: AWS AppConfig rolls a configuration change back when that change
triggers an Amazon CloudWatch alarm. The limit is that a flag only decides which code path runs
from now on, so data that the new path already wrote and messages it already sent stay as they
are.

## Why choose / why not
- Choose a flag when: a risky feature or a new dependency needs an off switch that is faster than
  a deploy or a rollback, and the change sits behind one decision point.
- Don't rely on a flag when: the change is a schema or message-format change; turning the code
  path off does not change the schema, so the change still needs a compatibility window.
- Budget for: the carrying cost, meaning more conditional paths to test and flags that must be
  removed once the feature is stable.

## Interview angle
- Probed as "what is the fastest way to undo a bad release?"; a flag, if the feature is behind
  one, then traffic switching, then a rollback.
- Common wrong answer: "with feature flags we never need rollbacks."
- Strong answer: name the kill switch as the first step of an incident, then state the limit: the
  flag switches behaviour, not data.

## Related
- [[Expand and contract schema changes keep the previous version runnable after a rollback]]: a
  flag switches code paths but not the schema, so a schema change still needs that window.
- [[Blue-green switches all traffic at once while a canary shifts a subset of users first]]: those
  strategies undo a release by moving traffic between versions; a flag undoes a feature inside
  one version.
