---
tags: [java, java-core, records, jpa, interview]
status: draft
author: claude
up: ["[[Java core MOC]]", "[[Spring MOC]]"]
source: "https://jakarta.ee/specifications/persistence/3.1/jakarta-persistence-spec-3.1.html#a18"
created: 2026-10-07
review: unjudged
---
# A record cannot be a JPA entity, because the provider must subclass it, construct it empty and write its fields later

## Core idea
Jakarta Persistence 3.1, the version Spring Boot 3.2.1 manages, requires an entity class to be
non-final, with no final methods or persistent fields and a no-argument constructor; version 3.2
names records as not allowed. A record fails each rule for a reason the provider cares about. It is
implicitly `final`, so Hibernate cannot subclass it with Byte Buddy to build a lazy-loading proxy.
It has no no-argument constructor unless it declares one, and that one must delegate to the
canonical constructor. `Field.set` on a record field throws `IllegalAccessException` even after
`setAccessible(true)`, while the same call on a final field of an ordinary class succeeds on
OpenJDK 23.0.2. And a record cannot change: a "change" is a new instance that the persistence
context does not track.

## Why choose / why not
- Use a record when: the type is a query result, such as a Hibernate 6.4 selection query, `select
  new`, or a Spring Data JPA class-based DTO projection; the results are not managed entities.
- Use a record as an embeddable when: Hibernate 6.4 or Jakarta Persistence 3.2 is in use; the 6.4
  documentation still rules out a record as `@EmbeddedId`, and 3.2 makes record primary-key classes
  standard.
- Don't use a record as an entity when: lazy loading, dirty checking or setters matter; keep the
  entity an ordinary class and use the record for the projection or the embeddable. → see
  [[An advised final class fails Spring Boot startup because CGLIB cannot subclass it]]
- Don't expect record equality to suit an entity: it covers all components and changes whenever any
  value changes, while an entity usually needs equality that stays the same for one database row (a design need,
  not a rule of Jakarta Persistence 3.1).

## Interview angle
- Asked as "can a record be a JPA entity?" or "why can't Hibernate use a record?".
- Common wrong answer: "I put `@Entity` on my records."
- Strong answer: cite the rule (3.1: non-final class and no-argument constructor; 3.2: records
  named), then the mechanics (no subclass for the proxy, no empty constructor, `Field.set`
  refuses), then where records do fit: projections and embeddables.

## Related
- [[An advised final class fails Spring Boot startup because CGLIB cannot subclass it]]: the same
  "final means no subclass" limit seen from Spring AOP, where a record also fails; this note adds
  the provider's lazy-loading proxy, the missing no-argument constructor and the unwritable fields.
- [[Java core MOC]]: records belong to the modern-Java cluster, and this note marks where they stop
  working.
- [[Spring MOC]]: Spring Data JPA projections are one place in the Spring stack where a record does
  fit.
- Written up in win-interview: backend/java/docs/modern-java.md, section 2.2.5
