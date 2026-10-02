---
tags: [java, spring, transactions, isolation, interview]
status: draft
author: claude
up: ["[[@Transactional MOC]]"]
source: "https://docs.spring.io/spring-framework/docs/6.1.x/javadoc-api/org/springframework/transaction/TransactionDefinition.html"
created: 2026-10-01
score: 0.867
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Without an isolation attribute, a Spring transaction runs at the database's default isolation level

## Core idea
`@Transactional` defaults to `Isolation.DEFAULT`, which Spring's `TransactionDefinition` defines
as using the default isolation level of the underlying datastore. Spring then leaves the JDBC
connection's isolation level as the database set it, so the same method runs at READ COMMITTED
on PostgreSQL but at REPEATABLE READ on MySQL InnoDB. A non-default value, such as
`isolation = Isolation.SERIALIZABLE`, is set on the JDBC connection when the transaction begins,
and Spring resets the connection's previous level after the transaction, so the pooled
connection does not keep it. A lab on Spring Framework 6.1.2 with Hibernate 6.4.1 and H2
confirmed both: without the attribute the transaction's connection reported H2's default READ
COMMITTED, and with `SERIALIZABLE` it reported `TRANSACTION_SERIALIZABLE`.

## Why choose / why not
- Keep the default when: the code runs on one database and was written and tested for that
  database's default level; an explicit level then adds nothing.
- Set an explicit level when: the code relies on a level's guarantees and may run on databases
  with different defaults, such as one tested on PostgreSQL that also ships on MySQL; otherwise
  the same method allows different anomalies on each.
- Don't raise the isolation of a whole method to fix one race: a `@Version` column, a targeted
  `SELECT ... FOR UPDATE` or a conditional `UPDATE` usually fixes it with less blocking and fewer
  aborted transactions.

## Interview angle
- Probed as "what isolation level does `@Transactional` use?".
- Common wrong answer: "Spring uses READ COMMITTED" or "SERIALIZABLE, to be safe".
- Strong answer: `Isolation.DEFAULT` hands the choice to the database, whose default differs by
  vendor; an explicit level is set on the connection only where the transaction starts, and reset
  afterwards.

## Related
- [[A joined REQUIRED transaction ignores its own isolation, timeout and readOnly attributes]]:
  partial overlap; that note covers the inner method whose level is ignored, this one covers which
  level the starting method gets when it sets none.
- [[MySQL InnoDB REPEATABLE READ applies UPDATE and DELETE to the latest committed rows, not to the snapshot]]:
  what the default level means on MySQL InnoDB, where the same Spring code reads and writes
  differently than on PostgreSQL.
- [[A version column detects a lost update at write time instead of blocking the other writer]]:
  the cheaper fix that the third bullet prefers over raising the isolation level.
