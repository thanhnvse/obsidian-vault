---
tags: [java, java-core, string, performance, interview]
status: draft
author: claude
up: ["[[Java core MOC]]"]
source: "https://docs.oracle.com/javase/specs/jls/se21/html/jls-15.html#jls-15.18.1"
created: 2026-10-07
review: unjudged
---
# String += in a loop copies everything built so far on every iteration, and invokedynamic in Java 9 did not change that

## Core idea
Every `+` that is not a constant expression creates a new `String` (JLS 15.18.1). Since JDK 9
javac emits one `invokedynamic` per expression, linked by `StringConcatFactory` (JEP 280), and
before that a `StringBuilder` chain; that changed how each `+` runs, not how many run. In
`result += part` inside a loop, each iteration allocates an array as long as the whole result so
far and copies it, so the cost is quadratic in the number of parts. On OpenJDK 23.0.2, 1,000 parts
of 10 characters allocated 5,047,968 bytes with `+=` and 47,088 with a `StringBuilder`; with
2,000 parts the `+=` figure grew by a factor of 3.98 and the builder's by 2.00. A builder grows
its buffer to about twice its size when full, so copying stays linear.

## Why choose / why not
- Keep `+` when: it is one expression such as `"id=" + region + ":" + id`. The JDK sizes and
  allocates the result once (on 23.0.2 for up to 20 non-constant arguments, on 21 always), so a
  hand-written builder adds nothing.
- Use `StringBuilder` when: you append in a loop; `new StringBuilder(expectedLength)` avoids
  regrowing the buffer when the final size is known.
- Use `String.join`, `StringJoiner` or `Collectors.joining` when: you only need a delimiter between
  parts; they are linear too.
- Don't reach for `StringBuffer` to fix a loop: it only adds a lock per call, and most builders
  are never shared between threads.

## Interview angle
- Probed as "what does javac generate for `+`, and do we still need `StringBuilder`?"
- Common wrong answers: "since Java 9, `+` in a loop is optimised", and its opposite, "always use
  `StringBuilder` instead of `+`."
- Strong answer: constants are folded; otherwise one `invokedynamic` and one new `String` per
  evaluation; a single expression needs no builder, a loop does because each iteration copies
  everything so far. Back it with a measured figure or `javap -c -v` output, not a claim.

## Related
- [[Java core MOC]]: the "what does the JVM actually do with this code?" question, applied to the
  compiler's translation of `+`.
- [[A lambda is linked through invokedynamic at run time, not compiled to its own class file]]:
  string concatenation is another `invokedynamic` call site javac emits, and it is linked once on
  first execution in the same way.
- [[String immutability comes from a private final byte array that no method changes or exposes, not from the class being final]]:
  every change to a `String` creates a new object, which is the cost this loop multiplies.
- Written up in win-interview: backend/java/docs/strings-and-immutability.md, sections 3.2 and 3.3
