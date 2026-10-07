---
tags: [java, java-core, inheritance, collections, interview]
status: draft
author: claude
up: ["[[Java core MOC]]", "[[Java collections MOC]]"]
source: ""
created: 2026-10-07
review: unjudged
---
# Overriding add and addAll in a HashSet subclass counts each addAll element twice, because the inherited addAll calls add

## Core idea
Bloch's `InstrumentedHashSet` (Effective Java, Item 18) overrides `add` to do `addCount++` and
`addAll` to add `c.size()` before calling `super.addAll`. On JDK 21, `addAll(List.of("a", "b", "c"))`
leaves `size()` at 3 and `addCount()` at 6. `HashSet` does not override `addAll`: it inherits
`AbstractCollection.addAll`, which adds each element by calling `this.add`, and that is the
subclass's override. The same two overrides on `ArrayList` count 3, because `ArrayList.addAll`
copies an array and never calls `add`. So whether a subclass is right depends on how its superclass
calls itself, which the `addAll` contract does not fix (`AbstractCollection.addAll` documents its self-use in an
`@implSpec`, but `HashSet` does not repeat it). That makes inheritance fragile: when a
superclass is changed to implement `addAll` as a loop over `add`, an unchanged subclass goes from 3
to 6 in the lab.

## Why choose / why not
- Compose and forward when: you add behaviour such as counting to a collection or to any class you
  do not control; an `InstrumentedSet` that wraps a `Set` counts once per element (3 for both
  `HashSet` and `LinkedHashSet`), whatever the wrapped set does inside.
- Inherit when: the subclass really is a kind of the superclass, in the same package or team, and the
  superclass documents which overridable methods it calls itself and is meant to be extended;
  otherwise make the class or its methods `final` (Bloch, Items 18 and 19; a judgement).
- Accept the cost of composition: every method is forwarded by hand (Guava ships a `ForwardingSet`),
  and the wrapper is not a `HashSet`, so it cannot be passed where one is required.

## Interview angle
- Asked as "composition over inheritance: why?".
- Common wrong answer: "Inheritance is for code reuse."
- Strong answer: walk the count: `HashSet.addAll` is inherited from `AbstractCollection` and calls
  `add`, so 3 becomes 6 while `ArrayList` is right; add that a superclass change can break an
  unchanged subclass; then the forwarding wrapper, and when inheritance is fine.

## Related
- [[Java collections MOC]]: the example uses `HashSet` and `ArrayList`, two of the collections that
  map covers.
- [[ArrayDeque replaces both Stack and LinkedList for stacks and queues]]: the JDK's own
  `Stack extends Vector` is a different failure of inheritance, where every `List` operation becomes
  public API and `stack.remove(0)` removes the bottom element; that note gives the `Deque`
  alternative.
- [[Java core MOC]]: the map entry for what a design choice costs once the JVM runs the code.
- Written up in win-interview: backend/java/docs/oop-and-solid.md, section 2.4
