---
tags: [java, spring, transactions, interview]
status: draft
author: claude
up: ["[[@Transactional MOC]]"]
source: "https://docs.spring.io/spring-framework/docs/6.1.x/javadoc-api/org/springframework/transaction/TransactionTimedOutException.html"
created: 2026-10-01
score: 0.867
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A @Transactional timeout fails the next database access after the deadline instead of interrupting the method

## Core idea
`@Transactional(timeout = n)` gives the transaction a deadline, `n` seconds after it begins.
Nothing interrupts the method's thread when that deadline passes. Spring's local transaction
strategies throw `TransactionTimedOutException` when the deadline has been reached at the moment a
data access operation is attempted. For a `JdbcTemplate` statement or a JPA query created through
Spring's shared `EntityManager`, Spring also applies the time left until the deadline as the
query timeout, so a slow query that starts in time can still be cut off. A method that spends its
whole timeout in Java code or waiting on a remote HTTP
call therefore finishes that work undisturbed, and only its next database access fails. A lab on
Spring Framework 6.1.2 with Hibernate 6.4.1 confirmed this: a method with `timeout = 1` slept for
1.5 seconds without being interrupted, and its next repository query failed with
`TransactionTimedOutException`.

## Why choose / why not
- Set a transaction timeout when: the risk is a slow query or a long lock wait inside the
  transaction; the time left becomes each statement's query timeout, so the database call itself
  is cut off.
- Don't rely on it to bound remote calls or CPU work inside the transaction: nothing interrupts
  them, so give every HTTP client its own connect and read timeouts, and keep remote calls out of
  the transaction.
- Put it on the method that starts the transaction: a `REQUIRED` method that joins an existing
  transaction keeps the outer deadline and ignores its own.

## Interview angle
- Probed as "the payment call inside my `@Transactional(timeout = 5)` method hung for a minute;
  why did the timeout not fire?".
- Common wrong answer: "the transaction timeout aborts the method once the time is up."
- Strong answer: it is a deadline that Spring checks when the transaction's resources are used
  and passes to each statement as a query timeout; Java code and remote calls run to completion,
  so they need timeouts of their own.

## Related
- [[A joined REQUIRED transaction ignores its own isolation, timeout and readOnly attributes]]:
  the deadline described here is set only where a physical transaction begins; a joining
  method's own timeout is ignored.
- [[RestClient and WebClient take their timeouts from the underlying HTTP library]]: the remote
  call that this deadline does not interrupt has to be bounded on its HTTP client instead.
