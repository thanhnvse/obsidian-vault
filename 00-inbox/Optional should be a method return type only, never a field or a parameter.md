---
tags: [java, java-core, optional, interview]
status: draft
author: claude
up: ["[[Java core MOC]]"]
source: "https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Optional.html"
created: 2026-10-07
review: unjudged
---
# Optional should be a method return type only, never a field or a parameter

## Core idea
The `Optional` javadoc says it is primarily intended as a method return type where there is a clear
need to represent "no result" and `null` would likely cause errors. As a field it breaks Java
serialisation: `Optional` is not `Serializable`, so a `Serializable` class with an `Optional` field
fails with `NotSerializableException: java.util.Optional`, whether or not a value is present. As a
parameter it adds a state: the caller can still pass `null`, so the method has three cases instead
of two, and `greeting(null)` throws `NullPointerException` inside. A blind `get()` on an empty
`Optional` throws `NoSuchElementException: No value present`; `orElseThrow()`, added in Java 10, is
the same call with an honest name.

## Why choose / why not
- Return `Optional<T>` when: a lookup may find nothing and absence is a real business case, such as
  an optional nickname; the signature forces the caller to choose `orElse`, `orElseGet`,
  `orElseThrow` or `map`.
- Return an empty list when: the result is a collection; an empty list already means "no results",
  and `Optional<List<T>>` adds a wrapper for nothing.
- Use a nullable field when: the object goes through Java serialisation, such as HTTP session
  replication; offer an `Optional` getter only where absence is meaningful.
- Overload the method when: a parameter is optional; an `Optional` parameter still accepts `null`.
- Use `orElseGet(this::compute)` when: the default is costly; `orElse(compute())` runs `compute()`
  even when a value is present.

## Interview angle
- Asked as "how should `Optional` be used?" or "is `Optional` a replacement for `null`?".
- Common wrong answer: "`Optional` replaces null everywhere", or "`isPresent()` then `get()` is the
  proper way", which is a `null` check with more steps.
- Strong answer: the return type of a lookup that may find nothing; not a field (not `Serializable`),
  not a parameter (three states), not around a collection, never a blind `get()`; quote the
  javadoc's "primarily intended" API note.

## Related
- [[Java core MOC]]: the map for Java language and library behaviour interviewers probe; this note
  is its entry on how to model a missing value.
- Written up in win-interview: backend/java/docs/modern-java.md, section 2.8
