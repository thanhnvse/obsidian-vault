---
tags: [moc, java, spring, transactions, interview]
type: moc
status: draft
author: claude
up: ["[[Spring MOC]]"]
created: 2026-09-30
---
# @Transactional MOC

The question behind this map: *when does a transaction start, roll back, or silently not exist?*

## Rollback, propagation and proxies
- [[A checked exception commits a Spring @Transactional method by default]]: the rollback rule interviewers probe first
- [[Self-invocation bypasses the Spring @Transactional proxy]]: the proxy consequence behind most "it didn't work" stories
- [[Catching an exception from a joined REQUIRED method ends in UnexpectedRollbackException]]: shared fate under REQUIRED
- [[A joined REQUIRED transaction ignores its own isolation, timeout and readOnly attributes]]: attributes apply only where a transaction starts
- [[REQUIRES_NEW keeps the outer connection while it borrows a second one from the pool]]: the cost of independence
- [[readOnly = true on a JPA transaction silently drops changes to managed entities]]: what readOnly does, and what it does not
- [[@TransactionalEventListener moves a side effect after the commit but loses it if the process dies]]: side effects at the edge of the transaction (parked: title holds two ideas)

## Lifecycle
- [[@Transactional does not apply inside @PostConstruct because the proxy is created after initialisation]]: the lifecycle case: no proxy yet, so no transaction
