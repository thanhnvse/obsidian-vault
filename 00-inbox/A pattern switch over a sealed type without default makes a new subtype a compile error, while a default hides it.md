---
tags: [java, java-core, sealed-types, pattern-matching, interview]
status: draft
author: claude
up: ["[[Java core MOC]]"]
source: "https://openjdk.org/jeps/441"
created: 2026-10-07
review: unjudged
---
# A pattern switch over a sealed type without default makes a new subtype a compile error, while a default hides it

## Core idea
With `sealed interface PaymentResult permits Approved, Declined, Pending`, a `switch` that has one
case per subtype and no `default` is exhaustive: javac compares the case labels with the permitted
subclasses. Add a fourth permitted type and every such switch stops compiling
(`compiler.err.not.exhaustive` for a switch expression), which lists the places to change. A
`default` branch keeps compiling and sends the new type down the generic path. JEP 441 says an
exhaustive switch without a match-all clause is better than one with it. javac also compiles in
a synthetic default that throws `MatchException` at run time, so a switch compiled before the subtype was added,
and not recompiled, throws it when the new subtype arrives.

## Why choose / why not
- Leave out `default` when: the sealed hierarchy is yours, so every switch can be recompiled; the
  compiler then lists each one that needs the new case.
- Keep `default` when: the selector is not sealed (`Object`, `String`), or the hierarchy belongs to
  another team and a new subtype has a safe generic handling.
- Prefer an abstract method on the type when: new types arrive more often than new operations; a
  sealed switch makes a new operation one method, but every new type touches every switch.
- Recompile against the new version when: a library adds a subtype to a sealed interface you switch
  over; otherwise the old switch throws `MatchException`.

## Interview angle
- Asked as "why is a switch without `default` better than one with it?" or "what are sealed
  classes for?".
- Common wrong answer: "Always add a `default` to be safe", which removes the exhaustiveness check.
- Strong answer: a sealed type with records is a closed set the compiler knows, so no `default`
  turns a new subtype into a list of compile errors; then name the `MatchException` safety net for
  separate compilation.

## Related
- [[A lambda is linked through invokedynamic at run time, not compiled to its own class file]]: javac
  also compiles a pattern switch to an `invokedynamic` call (`SwitchBootstraps.typeSwitch`), so that
  note shows how such a call site is linked at run time, by a bootstrap method (`LambdaMetafactory` there).
- [[Java core MOC]]: sits beside the lambda and stream notes as the Java 21 language side.
- Written up in win-interview: backend/java/docs/modern-java.md, sections 2.3.2 and 2.3.3
