---
tags: [java, java-core, string, string-pool, interview]
status: draft
author: claude
up: ["[[Java core MOC]]"]
source: "https://docs.oracle.com/javase/specs/jls/se21/html/jls-3.html#jls-3.10.5"
created: 2026-10-07
review: unjudged
---
# String == is true only for pooled instances, and an effectively final variable does not make a concatenation a constant

## Core idea
`==` on two `String`s compares references (JLS 15.21.3), so two separately built equal strings are
`==` only when both are the pooled instance. Literals and constant expressions are
interned, so equal literals are one object in the whole JVM; `new String("Hello")` is always a
fresh object, `intern()` returns the pooled one, and a string computed at run time (decoded from a
request, a file or a database row) is not pooled. javac folds a concatenation into a pooled literal
only when every operand is a constant: a literal, or a constant variable (a `final` `String` or primitive
initialised with a constant expression, JLS 4.12.4). With `final String hel = "Hel"`, `(hel + "lo") == "Hello"` is `true`. With
`String hel = "Hel"` that is never reassigned (effectively final), or a `final` one set by a call
such as `String.valueOf("Hel")`, it is `false`: neither is a constant, so the `+` runs at run time.

## Why choose / why not
- Compare with `equals` when: the string comes from outside the class file. `"PAID".equals(status)`
  also gives `false` instead of a `NullPointerException` when `status` is null, and
  `Objects.equals(a, b)` is the null-safe form when either side may be null.
- Don't trust `==` when: a unit test passes because both sides are literals. It fails in
  production as soon as one side comes from a request or a database, as a branch that never fires
  although the data looks right in the log.
- Use `intern()` only when: duplicates that stay alive are the memory problem, since each call is a
  lookup in a native hash table. A scoped canonicalising map, or `-XX:+UseStringDeduplication`
  (which shares the arrays, not the objects, so `==` still does not hold), are the alternatives.

## Interview angle
- Probed as "how many objects does `new String("abc")` create?", then the trap pair
  `"Hel" + "lo" == "Hello"` against `hel + "lo" == "Hello"`.
- Common wrong answer: "`==` works for strings; my test passes", or that effectively final is as
  good as `final` for folding.
- Strong answer: `==` compares references and `equals` compares characters (and already returns at
  once for the same object); literals and constants are pooled (JLS 3.10.5) and run-time strings
  are not; folding needs the keyword `final` plus a constant initialiser.

## Related
- [[Java core MOC]]: the "what does the JVM actually do with this code?" question, for the String
  comparison interviewers open with.
- [[String immutability comes from a private final byte array that no method changes or exposes, not from the class being final]]:
  why one pooled instance can stand for every equal literal.
- [[A lambda captures the value of a local variable, which is why the variable must be effectively final]]:
  the same term, effectively final, appears there for a different rule; here it is exactly why a
  concatenation does not fold.
- Written up in win-interview: backend/java/docs/strings-and-immutability.md, sections 2.3 to 2.5
