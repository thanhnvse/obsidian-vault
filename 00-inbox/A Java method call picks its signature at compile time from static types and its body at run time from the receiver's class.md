---
tags: [java, java-core, polymorphism, oop, interview]
status: draft
author: claude
up: ["[[Java core MOC]]"]
source: "https://docs.oracle.com/javase/specs/jls/se21/html/jls-15.html#jls-15.12.2"
created: 2026-10-07
review: unjudged
---
# A Java method call picks its signature at compile time from static types and its body at run time from the receiver's class

## Core idea
A call such as `b.m("text")` is resolved in two steps. At compile time javac looks up `m` in the
static type of the receiver and picks the most specific applicable signature from the static types
of the arguments (JLS 15.12.2); the class file records that type and signature, such as
`invokevirtual Base.m:(Ljava/lang/Object;)Ljava/lang/String;`, with no mention of the runtime
class. At run time the JVM starts in the receiver object's class and walks up to find the body for
that signature. With `Base b = new Derived()`, where `Derived` adds an overload `m(String)` and
overrides `m(Object)`, `b.m("text")` runs `Derived.m(Object)`. Overloads are decided by step one
alone: `print(Animal)` runs for an `Animal`-typed variable that holds a `Dog`. `static` methods,
`private` methods and fields skip step two.

## Why choose / why not
- Override when: behaviour must depend on the object's runtime class; the body is picked from the
  receiver.
- Don't overload when: you expect the runtime class of an argument to choose the method; only its
  static type does.
- Always write `@Override`: `equals(Money other)` is an overload of `equals(Object)`, so
  `ArrayList.contains` calls `Object.equals` and misses, and `@Override` turns that into a compile
  error. → see [[Overriding equals without hashCode makes HashMap lookups miss]]
- Don't treat `static` methods or fields as overridable: they are hidden and chosen by the static
  type, and a `static` call through a `null` reference throws no `NullPointerException`.

## Interview angle
- Asked as "overloading vs overriding?".
- Common wrong answer: "Overloading is runtime polymorphism", or "a `static` method can be
  overridden".
- Strong answer: show `Base b = new Derived(); b.m("text")`, name step one at compile time and step
  two at run time, then say what skips dispatch: `static` methods, `private` methods and fields.

## Related
- [[List.remove(1) on a List-Integer- removes the element at index 1, not the value 1]]: that note
  applies the overload phases (strict, boxing, varargs) to one JDK method; this one is the general
  two-step rule, and adds that the receiver's class never changes the signature that was chosen.
- [[Overriding equals without hashCode makes HashMap lookups miss]]: that note covers a lookup that
  misses for want of `hashCode`; an `equals(Money)` overload is another way an intended override is
  never called by a collection.
- [[Java core MOC]]: the map entry for what the compiler decides and what the JVM decides.
- Written up in win-interview: backend/java/docs/oop-and-solid.md, section 2.2
