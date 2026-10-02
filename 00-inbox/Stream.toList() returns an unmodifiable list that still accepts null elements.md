---
tags: [java, java-core, streams, interview]
status: draft
author: claude
up: ["[[Java core MOC]]"]
source: "https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Stream.html#toList()"
created: 2026-10-01
score: 0.84
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Stream.toList() returns an unmodifiable list that still accepts null elements

## Core idea
The JDK 21 Javadoc says the list returned by `Stream.toList()` is unmodifiable: calls to any mutator
method always throw `UnsupportedOperationException`. The same Javadoc specifies the default
implementation as if by `Collections.unmodifiableList(new ArrayList<>(Arrays.asList(this.toArray())))`,
a wrapper around a copy, and that copy keeps `null` elements. On OpenJDK 21,
`Stream.of("a", null).toList()` returns a two-element list whose second element is `null`. So the
list cannot be changed, but it does not reject `null` the way some other unmodifiable lists do.

## Why choose / why not
- Use `toList()` when: the result is handed on and should not change, such as a DTO list returned
  from a service; it is shorter than `collect(Collectors.toList())` and states the intent.
- Use `collect(Collectors.toCollection(ArrayList::new))` when: the caller must add, remove or sort
  the result in place; `Collectors.toList()` happens to be mutable on OpenJDK, but its Javadoc makes no
  guarantees on the type, mutability, serializability or thread-safety of the list.
- Use `Collectors.toUnmodifiableList()` when: a `null` element should fail fast; that collector
  throws `NullPointerException` on a `null` value, like `List.of`, while `toList()` keeps it.

## Interview angle
- Probed as "what is the difference between `Stream.toList()` and `collect(Collectors.toList())`?"
- Common wrong answer: "none, `toList()` is just shorter."
- Strong answer: `toList()` is unmodifiable and still allows `null`; `Collectors.toList()` promises
  nothing about mutability; switching a call site from one to the other breaks code that later adds
  to the list.

## Related
- [[Collectors.toMap throws on a duplicate key unless you pass a merge function]]: another collection
  contract at the terminal operation that differs from what the equivalent loop would do.
- [[Arrays.asList returns a fixed-size view that writes through to the array]]: the specified default
  implementation of `toList()` copies through `Arrays.asList` and then wraps the copy, so unlike that
  view the result rejects `set` too.
