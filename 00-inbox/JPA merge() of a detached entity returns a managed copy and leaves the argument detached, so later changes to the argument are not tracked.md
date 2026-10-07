---
tags: [java, spring, jpa, hibernate, interview]
status: draft
author: claude
up: ["[[Spring MOC]]"]
source: "https://jakarta.ee/specifications/persistence/3.1/jakarta-persistence-spec-3.1.html#merging-detached-entity-state"
created: 2026-10-07
review: unjudged
---
# JPA merge() of a detached entity returns a managed copy and leaves the argument detached, so later changes to the argument are not tracked

## Core idea
`persist(x)` makes `x` itself managed. `merge(x)` copies the state of `x` onto a managed instance of
the same identity, creating one if needed, and returns that instance; Jakarta Persistence 3.1
describes the managed copy as `X'`. In a lab on Hibernate 6.4.1, `merge()` of a detached entity sent
one `SELECT` (none when the context already held the row), returned a different instance, and left
the argument detached, so a setter called on the argument afterwards was lost. Spring Data's `save()`
chooses between the two: `persist()` when it considers the entity new (a `null` id, or a `null`
`@Version` if the entity has one), `merge()` otherwise, so `save(detached)` also returns a copy. Only
the returned instance is managed. `merge()` copies every loaded field, `null` included.

## Why choose / why not
- Choose `merge()` when: you hold a detached entity, for example one built in an earlier request, and
  want its state written; keep the returned instance and drop the argument.
- Don't merge an object built from a request body for a `PATCH`: a field the client left out is
  `null`, and the merge overwrote the column with `NULL` in the lab, which is a `PUT` in effect. Load
  the entity with `find()`, set only the fields the request names, and let dirty checking write them.
- Don't call `persist()` on a detached entity: with a generated id it throws `EntityExistsException`
  at once and the transaction can no longer commit; with an assigned id and no `@Version`, Hibernate
  takes it for a new object (unless the second-level cache holds that id), and the `INSERT` fails at
  flush with a duplicate key.
- Don't rely on how an id with no row behaves: on 6.4.1 `merge()` inserted a new row under a new id,
  and the 6.6 migration guide says that case changed (not run in the lab).

## Interview angle
- Asked as "`persist` vs `merge`?", or "does `merge()` reattach the object?".
- Common wrong answer: "`merge()` reattaches the object", or "`persist()` and `merge()` are the same
  for new objects".
- Strong answer: `persist` manages the argument and fails on a detached entity; `merge` copies state
  onto a managed instance and returns it while the argument stays detached; so use the return value,
  expect nulls to be copied, and say which Hibernate version you checked.

## Related
- [[Spring MOC]]: Spring Data's `save()` is where most Spring code meets this choice, because it
  silently picks `persist` or `merge` by whether the entity looks new.
- [[readOnly = true on a JPA transaction silently drops changes to managed entities]]: the other half
  of "do I need `save()`?"; a managed entity is written by dirty checking, while a detached one is not
  tracked at all, which is the gap `merge()` closes.
- [[Catching an exception from a joined REQUIRED method ends in UnexpectedRollbackException]]: a failed
  `persist(detached)` marks the transaction rollback-only in the same way, so catching its exception
  does not save the commit.

Written up in win-interview: backend/java/docs/jpa-persistence-context.md, sections 2.5 and 2.6
