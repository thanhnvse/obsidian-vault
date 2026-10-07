---
tags: [java, java-core, solid, liskov, interview]
status: draft
author: claude
up: ["[[Java core MOC]]"]
source: "https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html#jls-8.4.8.3"
created: 2026-10-07
review: unjudged
---
# The compiler checks an override's signature but not its behaviour, so a subtype can compile and still break Liskov substitution

## Core idea
Liskov substitution says that whatever is provable about a `T` should hold for an `S` that is a
subtype of `T`: a subtype may accept more, must promise at least as much, and must keep the
supertype's invariants. javac checks only the signature half: an override keeps the parameter types,
returns a compatible type and adds no checked exception (JLS 8.4.8.3). Behaviour is unchecked. A
caller written against `Rectangle` calls `setWidth(5)` and `setHeight(4)` and returns `area()`: 20
for a `Rectangle`, 16 for a `Square` whose setters keep both sides equal, and nothing fails to
compile. A `LimitedAccount.withdraw` that rejects amounts above 100 breaks a caller that drains 500
from an `Account`.

## Why choose / why not
- Ask it as a review question when: you write any `extends` or `implements`; it is a check, not a
  structure to build.
- Don't subclass when: the type cannot honour the supertype's contract; compose instead of building
  an elaborate hierarchy.
- Take the mutation out of the shared type when: `Square` and `Rectangle` clash; immutable records
  that share only `area()` fix it.
- Put read-only in the type when: you design an API; `List.of(...).add(...)` compiles and throws
  `UnsupportedOperationException`, a documented optional operation, so a method that only reads
  should accept `Collection<? extends E>` or `Iterable<E>`.

## Interview angle
- Asked as "explain Liskov with an example".
- Common wrong answer: "A `Square` is a `Rectangle`, so it extends it."
- Strong answer: show the caller, not just the classes (the 20 against 16); add a JDK example such
  as `List.of(...).add`; say it is a behavioural contract the compiler cannot check, and that the
  smell is a subclass throwing `UnsupportedOperationException` from an inherited method.

## Related
- [[Arrays.asList returns a fixed-size view that writes through to the array]]: a concrete `List`
  that allows `set` but throws on `add`; this note puts that behaviour in the Liskov frame, where
  callers code to what the `List` contract promises about optional operations.
- [[Java core MOC]]: the map entry for what a type promises beyond its signatures.
- Written up in win-interview: backend/java/docs/oop-and-solid.md, section 3.3
