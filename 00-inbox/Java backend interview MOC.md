---
tags: [moc, java, interview]
type: moc
status: draft
author: claude
created: 2026-09-30
---
# Java backend interview MOC

Entry point for the Java backend interview topics. Each area opens with the one question the
interviewer is really asking. Long-form write-ups with runnable proof live in the
`win-interview` repository; these notes are the atomic claims to rewrite in your own words.

Sources cite the documentation version current on 2026-09-30 (for example Spring Framework 7.0,
Kafka 4.3, Kubernetes 1.37). Check each against the version your target team runs.

## Maps
- [[Java core MOC]]: what does the JVM actually do with this code?
- [[Spring MOC]]: what does the container do to my bean, and when?
- [[Database MOC]]: what does the database guarantee, and what does it cost?
- [[Concurrency MOC]]: what happens when two requests touch the same data at once?
- [[Security MOC]]: who can read, forge or replay this?
- [[Microservices and messaging MOC]]: what happens when the other side is slow, down, or sees the message twice?
- [[System design MOC]]: which load dominates, and what gives way first?
- [[Ops and cloud MOC]]: how does this ship, and how does it come back when it breaks?

## Open questions
- `atomic` fails most often on notes that compare two options. Decide whether comparisons become two notes, or whether the rubric question should change (a rubric change needs a new version and a re-run).
- Not yet captured: N+1 queries, "no network I/O inside a transaction", HS256 vs RS256 (stopped at the privacy gate), consumer rebalancing, the generational hypothesis.
