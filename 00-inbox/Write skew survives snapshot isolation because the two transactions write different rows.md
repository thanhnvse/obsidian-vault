---
tags: [database, sql, transactions, isolation, concurrency, postgresql, interview]
status: draft
author: claude
source: "https://www.postgresql.org/docs/18/transaction-iso.html"
created: 2026-09-30
score: 0.871
review: "borderline"
score_reasons: ["atomic: 0.54 (borderline)", "numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 2
---
# Write skew survives snapshot isolation because the two transactions write different rows

## Core idea
Snapshot isolation gives each transaction one consistent snapshot and stops it with a conflict
only when it tries to change a row that a concurrent transaction has already changed. Write
skew slips past that check: two transactions read an overlapping set of rows, both see that a
rule holds, and each then writes a different row. With the rule "at least one doctor stays on
call", two transactions each count two doctors on call and each take only their own doctor off
call; no row is written twice, so both commit and nobody is on call. PostgreSQL 18 implements
REPEATABLE READ as snapshot isolation, so its REPEATABLE READ lets this anomaly through.

## Why choose / why not
- Use SERIALIZABLE when: a rule spans several rows, such as the on-call minimum, and the code
  can retry the whole transaction on SQLSTATE `40001`; PostgreSQL 18 SERIALIZABLE tracks the
  read/write dependencies and aborts one of the two transactions.
- Lock the rows the rule reads instead when: retries are expensive;
  `SELECT id FROM doctors WHERE on_call FOR UPDATE` makes the second transaction wait for the
  first rather than decide on a stale count.
- Don't rely on row locks when: the rule is about rows that do not exist yet, such as "no two
  bookings overlap"; there is nothing to lock, so use SERIALIZABLE or a constraint that states
  the rule.

## Interview angle
- Probed as "we check that one doctor stays on call before updating; why can both go off call
  under REPEATABLE READ?"
- Common wrong answer: "REPEATABLE READ, being snapshot isolation, is as safe as SERIALIZABLE."
- Strong answer: define write skew as reads of overlapping rows followed by writes to different
  rows, then give the three fixes: SERIALIZABLE with retry, a lock on what the rule reads, or a
  constraint.

## Related
- [[READ COMMITTED gives each statement its own snapshot, so two reads in one transaction can disagree]]:
  REPEATABLE READ removes that per-statement snapshot, yet write skew survives it, so a stable
  snapshot alone does not protect a rule that spans rows.
