---
tags: [java, java-core, string, immutability, interview]
status: draft
author: claude
up: ["[[Java core MOC]]"]
source: "https://github.com/openjdk/jdk23u/blob/jdk-23.0.2-ga/src/java.base/share/classes/java/lang/String.java#L144-L182"
created: 2026-10-07
review: unjudged
---
# String immutability comes from a private final byte array that no method changes or exposes, not from the class being final

## Core idea
On OpenJDK 23.0.2 a `String` keeps its characters in a `private final byte[] value`, next to a
`coder` flag and a cached `hash`, and no public method changes that array or hands it out. The
class is also `final`, which stops a subclass from adding state or overriding methods; both are
needed, but the array that never changes is what fixes the value. Four things are safe because of it: sharing
one instance (the string pool), computing `hashCode()` once and caching it in `hash`, passing a
string between threads without locks (the final-field guarantee of JLS 17.5), and trusting a
checked value such as a file path after the check. The price is that every change creates a new
object, and a `String` cannot be wiped after use, so the JCA guide has `PBEKeySpec` take a `char[]`.

## Why choose / why not
- Copy the design for your own value classes when: a value is shared, used as a map key or handed
  to other threads: a `final` class, `private final` fields, mutable inputs copied in, and nothing
  mutable handed out. `final` alone pins the reference, not the object behind it.
- Use a `StringBuilder` when: many changes build one string, because each change to a `String` is a
  new object → see [[String += in a loop copies everything built so far on every iteration, and invokedynamic in Java 9 did not change that]].
- Use a `char[]` when: the value is a password. A `char[]` can be cleared as soon as it has been
  used, while a `String` can stay in the heap, and so in heap dumps, until the collector reclaims
  and reuses the memory; this shortens the window and does not close it.

## Interview angle
- Probed as "why is `String` immutable?", followed by "the `hash` field is neither `final` nor
  `volatile`, so how is caching it thread-safe?"
- Common wrong answer: "because the class is `final`."
- Strong answer: name the private final array that no method changes or exposes, derive the four
  benefits, then state the cost. For the hash, it is a benign data race: every thread that
  races computes the same number from the immutable array.

## Related
- [[Java core MOC]]: the String questions sit under its "what does the JVM actually do with this
  code?" question, and this note is the mechanism the other String notes rely on.
- [[A HashMap key whose hashCode changes after put cannot be found until its hash changes back]]:
  that note shows what a mutable key does inside a map; a String's array and cached hash never
  change, which is why it is the safe key.
- [[String == is true only for pooled instances, and an effectively final variable does not make a concatenation a constant]]:
  one pooled instance can stand for every equal literal only because the array never changes.
- Written up in win-interview: backend/java/docs/strings-and-immutability.md, sections 2.1 and 2.2
