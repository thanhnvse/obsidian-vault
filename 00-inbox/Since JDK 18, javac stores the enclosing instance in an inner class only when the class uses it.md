---
tags: [java, java-core, lambda, interview]
status: draft
author: claude
up: ["[[Java core MOC]]"]
source: "https://www.oracle.com/java/technologies/javase/18-relnote-issues.html"
created: 2026-10-01
score: 0.887
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 2
---
# Since JDK 18, javac stores the enclosing instance in an inner class only when the class uses it

## Core idea
Before JDK 18, javac gave every inner class, anonymous classes included, a private synthetic field
named like `this$0` that holds a reference to the enclosing instance, even when the class never used
it. The JDK 18 release notes say that, starting in JDK 18, unused `this$` fields are omitted and the
field is generated only for inner classes that reference their enclosing instance. The same notes say
that subclasses of `java.io.Serializable` are not affected, so a serializable inner class still keeps
the field. The notes add that the change may let enclosing instances be garbage collected sooner,
which avoids a source of memory leaks when an inner class outlives its enclosing instance.

## Why choose / why not
- Rely on the change when: code is compiled with javac 18 or later and an anonymous listener or task
  never touches the outer object; it no longer keeps that object reachable.
- Don't rely on it when: the body calls an outer method or reads an outer field, even once; the field
  is generated and the outer object lives as long as the listener or the queued task.
- Don't rely on it when: the anonymous class is `Serializable`; it still holds the outer instance and
  also drags it into the serialized stream. A static nested class or a lambda holds only what it uses.

## Interview angle
- Probed as "can an anonymous inner class cause a memory leak?"
- Common wrong answer: "an anonymous class always holds a reference to its outer instance", or "never,
  the garbage collector handles it."
- Strong answer: it holds the outer instance through `this$0` only when it uses it, since JDK 18, and
  always if it is serializable; when it does, a long-lived listener keeps the whole outer object
  reachable, which shows in a heap dump as the outer class retained through `this$0`.

## Related
- [[A Java memory leak is an object that stays reachable after the program stops needing it]]: the
  `this$0` field is one concrete path that keeps an outer object reachable after the program stops
  needing it.
- [[Inside a lambda, this refers to the enclosing instance, not to the lambda]]: a lambda reaches the
  enclosing object through `this` only when its body uses it, which is the rule javac 18 brought to
  non-serializable inner classes.
