---
tags: [database, acid, cap, consistency, interview]
status: draft
author: claude
up: ["[[Database MOC]]"]
source: "https://sites.cs.ucsb.edu/~rich/class/cs293b-cloud/papers/brewer-cap.pdf"
created: 2026-10-07
review: unjudged
---
# ACID's C means a transaction preserves the database rules, while CAP's C is only single-copy consistency, a strict subset of ACID's

## Core idea
"Consistency" has three unrelated meanings. In ACID, the data satisfies every declared constraint
after each transaction (`NOT NULL`, `CHECK`, `UNIQUE`, foreign keys), and the
database enforces only what you declared: in a lab on PostgreSQL 15, a transfer that debited one
account and forgot to credit the other committed, because no constraint said "the total of all
balances never changes". In CAP, the C is single-copy consistency: every read sees the latest write,
as if there were one copy. Brewer's 2012 paper says it refers only to single-copy consistency, a
strict subset of ACID consistency. Eventual consistency, the third meaning, says only that replicas
converge if updates stop. An ACID system can still serve a stale read from an asynchronous replica.

## Why choose / why not
- Say "ACID's C" when: the question is about invariants inside one database; answer with what is
  declared as a constraint and what stays the application's job. Declare every rule you can, because
  a constraint holds for every writer, including the batch job and the SQL console.
- Say "CAP's C" when: the question is about replicas during a partition; name the choice per
  operation instead of calling the whole system "CP" or "AP".
- Don't read "not ACID" as "no guarantees": in an eventually consistent design each local transaction
  still has all four properties, and what is missing is the same guarantee across them.

## Interview angle
- Asked as "is the C in ACID the same as the C in CAP?", often right after "explain ACID".
- Common wrong answer: "yes, both mean the data is consistent", or "ACID means the database is
  always correct".
- Strong answer: ACID's C is the declared constraints; CAP's C is single-copy consistency; then one
  example of an ACID database whose replica returns a stale read, and what the CAP choice is for one
  concrete operation.

## Related
- [[Database MOC]]: ACID's C belongs to the question "what does the database guarantee?", with the
  limit that it guarantees only the rules you gave it.
- [[Read replicas scale reads but serve stale data while replication lags]]: the concrete case where a
  committed write on the primary is not yet visible on the copy, which is CAP's C, not ACID's.
- [[Read-your-writes on an asynchronous PostgreSQL replica needs recency routing, WAL-position routing or remote_apply]]:
  the fixes for the stale read a user gets after their own write, each with its own cost.
- [[A saga has no isolation, so concurrent sagas can read and overwrite each other's partly applied updates]]:
  what is left of ACID across services, where each local transaction keeps its properties but the whole
  operation has no isolation.

Written up in win-interview: backend/docs/acid.md, sections 2.2, 2.5 and 2.6
