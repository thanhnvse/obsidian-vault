---
tags: [java, spring, jpa, hibernate, interview]
status: draft
author: claude
up: ["[[Spring MOC]]"]
source: "https://jakarta.ee/specifications/persistence/3.1/jakarta-persistence-spec-3.1.html#a4374"
created: 2026-10-07
review: unjudged
---
# In FlushModeType.COMMIT Hibernate 6.4 runs a query without flushing first, so the query can miss the transaction's own pending changes

## Core idea
In the default `AUTO` mode, Hibernate flushes pending entity changes before a JPQL query whose tables
overlap them, so the query sees the changes. In `COMMIT` mode it flushes only at commit or on an
explicit `flush()`. Jakarta Persistence 3.1 leaves the effect of pending updates on queries
unspecified in that mode. In a lab on Hibernate 6.4.1 with `COMMIT` set, a JPQL query missed a
pending update, and a native query was not flushed either. A flush mode set on one
query wins over the mode of the persistence context: a `COMMIT` query in an `AUTO` context did not
flush, and an `AUTO` query in a `COMMIT` context did. With 50 managed entities and three queries,
`AUTO` checked 150 entities before the queries, `COMMIT` checked none, and the commit checked all 50 once.

## Why choose / why not
- Choose `COMMIT` on one hot query when: the persistence context is large, many queries do not depend
  on pending changes, and the dirty check before each `AUTO` flush is the cost you measured.
- Don't choose it when: the use case reads back through a query what it just changed; call `flush()`
  first, or stay on `AUTO`.
- Keep `AUTO` (the default) when unsure: it flushes only when a pending change touches the query's
  tables, but before a native query that names no query spaces it flushes everything (under the JPA
  bootstrap that Spring Boot uses).
- Know the strictest end of the dial: Spring's `readOnly` transactions use `MANUAL`, which never
  flushes, not even at commit.

## Interview angle
- Asked as "flush mode `AUTO` vs `COMMIT`?", or "why does my query not see the entity I just changed?".
- Common wrong answer: "a query always sees the changes of its own transaction", or "flush mode
  only decides when the commit happens".
- Strong answer: `AUTO` makes queries see pending changes, which Hibernate implements by table
  overlap; `COMMIT` skips the pre-query flush, so the query can miss them and the specification calls
  the result unspecified; a per-query mode wins; the cost of `AUTO` is a dirty check of every managed
  entity.

## Related
- [[Spring MOC]]: Spring Boot services use this dial through the JPA `EntityManager`, and
  `readOnly` changes it without telling you.
- [[readOnly = true on a JPA transaction silently drops changes to managed entities]]: `MANUAL` is the
  far end of the same flush-mode dial, where no flush ever happens and changes are dropped.
- [[A @Transactional test never commits, so it hides flush-time constraint errors, AFTER_COMMIT listeners and lazy-loading failures]]:
  flush is the moment a constraint error appears, so a test that never commits sees it only if it
  calls `flush()`.

Written up in win-interview: backend/java/docs/jpa-persistence-context.md, section 2.4
