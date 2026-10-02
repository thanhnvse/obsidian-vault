---
tags: [database, concurrency, race-condition, java, interview]
status: draft
author: claude
source: "https://docs.oracle.com/javase/specs/jls/se21/html/jls-17.html"
created: 2026-09-30
score: 0.925
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A synchronized block cannot prevent a lost update between two application instances

## Core idea
A lost update happens when two transactions read the same balance, each adds its amount in
application code, and the second write overwrites the first: two top-ups of 100 on a balance of
500 both read 500, both write 600, and the result is 600 instead of 700. Wrapping that
read-modify-write in `synchronized` excludes only threads that lock the same monitor, and every
monitor belongs to an object inside one JVM. Two instances of the service behind a load balancer
therefore hold two independent locks, and their requests still interleave on the same database
row. The exclusion has to live where the shared state lives, in the database.

## Why choose / why not
- Rely on a JVM lock only when: there is exactly one instance and the shared state is in its own
  memory, such as a local cache; it stops protecting anything once the state is a database row.
- Push the arithmetic into one statement when: the database can express it, such as
  `UPDATE accounts SET balance = balance + 100 WHERE id = ?`; under READ COMMITTED PostgreSQL
  applies the second update to the row version the first one committed, so no top-up is lost.
- Use a row lock or a version column when: the decision needs application logic between the
  read and the write, such as a credit-limit check in code; the database then serialises or
  rejects the competing write.

## Interview angle
- Probed as "we made the method `synchronized` and still lose top-ups in production"; the
  answer is that production runs more than one instance.
- Common wrong answer: fixing a database race with `synchronized` or a `ReentrantLock`.
- Strong answer: locate the shared state first, then put the exclusion there: an atomic
  `UPDATE`, a row lock, or a version check in the database.

## Related
- [[A Deployment treats its pods as interchangeable replicas of a stateless workload]]: every
  replica there runs its own JVM against state kept in a shared database, which is exactly the
  setup in which a JVM lock stops excluding anyone.
- [[READ COMMITTED gives each statement its own snapshot, so two reads in one transaction can disagree]]:
  under that default level the read and the later write see separate snapshots, which is the
  gap a lost update falls into.
