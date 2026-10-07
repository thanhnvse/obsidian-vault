---
tags: [java, hibernate, jpa, caching, interview]
status: draft
author: claude
up: ["[[Caching MOC]]"]
source: "https://docs.hibernate.org/orm/6.4/userguide/html_single/Hibernate_User_Guide.html#caching"
created: 2026-10-07
review: unjudged
---
# A write that bypasses Hibernate leaves its second-level cache serving the old value until the entry is evicted or expires

## Core idea
Hibernate's second-level cache belongs to the `SessionFactory`, so every transaction on it shares
the cache. It needs a cache provider (the lab used JCache with Caffeine), and entities opt in with
`@Cacheable`; a later `find()` is then served without a `SELECT`. Hibernate keeps the cache right
only for its own writes: with `READ_WRITE` the commit puts the new state into the entity region. The
guide warns that its caches are not aware of changes made to the persistent store by other
applications. In a lab on Hibernate 6.4.1, a plain JDBC `UPDATE` did not reach the cache: the next
`find()` returned the old name with no SQL, until `EntityManagerFactory.getCache().evict(...)` was
called. The entry stayed stale, and with a local provider such as Caffeine a second instance of your
own service counts as "other applications" too. The guide suggests a time-to-live retention policy on
the cache region, so that entries expire regularly.

## Why choose / why not
- Choose it when: the data is small, read far more often than it changes and loaded by id, such as
  countries or currencies, and every write goes through this application's Hibernate or a short
  staleness window is acceptable.
- Don't choose it when: batch jobs, JDBC code or other services write the table, or several
  instances each hold a local cache; use a clustered provider, a short expiry, or cache finished DTOs
  in a shared cache that you invalidate yourself.
- Keep the default when unsure: the guide recommends sticking to it, and by default entities are not
  part of the second-level cache.
- Don't expect the query cache to save you: any write to a table invalidates every cached query on
  that table, so it rarely pays off.

## Interview angle
- Asked as "first-level vs second-level cache? when would you use the second level?", or "users see
  different values on different nodes".
- Common wrong answer: "the second-level cache is safe; Hibernate keeps it in sync".
- Strong answer: scope (transaction vs `SessionFactory`), default (always on vs opt-in), and who
  keeps it fresh (only Hibernate's own writes); then treat it as a consistency decision first: who
  else writes this table, how many nodes, how much staleness is acceptable.

## Related
- [[Caching MOC]]: this adds the ORM-level cache to the map of what makes a cached value wrong, and
  for how long.
- [[Cache-aside loads data on a miss and leaves cache consistency to the application]]: the same rule
  one level up; a change made outside the cache's own write path is not seen until the item reloads,
  and an expiry bounds how stale it can get. Here the cache's own write path is Hibernate's, so only
  a write that Hibernate performs keeps the entry right.

Written up in win-interview: backend/java/docs/jpa-persistence-context.md, section 2.7
