---
tags: [java, concurrency, singleton, class-initialisation, interview]
status: draft
author: claude
up: ["[[Java core MOC]]", "[[Concurrency MOC]]"]
source: "https://docs.oracle.com/javase/specs/jls/se21/html/jls-12.html#jls-12.4.2"
created: 2026-10-07
review: unjudged
---
# The holder idiom stays thread-safe without synchronized because the JVM initialises the Holder class once, under its class-initialisation lock

## Core idea
In the holder idiom the singleton is a `static final` field of a private nested class `Holder`, and
`getInstance()` returns `Holder.INSTANCE`. JLS 12.4.2 gives every class an initialisation lock: the
first thread to read `Holder.INSTANCE` runs the static initialiser, and a thread that arrives
meanwhile waits until it finishes (step 2). The class is then labelled fully initialised and the
waiters are notified (step 10), so each of them gets the same, fully built instance, and the
constructor ran once. The idiom is lazy because `Holder` is a separate class: calling another static
method of the outer class does not initialise it. Once initialisation is done, the JVM may elide
(lược bỏ) the lock, but it must keep every happens-before ordering the lock would have given.

## Why choose / why not
- Choose when: plain Java, lazy creation, and a constructor that cannot fail; it is short, and the
  JVM does the locking, so there is no code to get wrong.
- Don't choose when: the constructor does I/O or can fail, such as loading settings from a config
  server. If it throws, the first caller gets `ExceptionInInitializerError`, the class is marked
  erroneous, and every later call gets `NoClassDefFoundError: Could not initialize class` until the
  JVM restarts; the constructor never runs again. DCL retries instead → see [[Since JDK 5, double-checked locking with non-final fields needs a volatile field so the lock-free first read sees the constructor's writes]].
- Don't choose when: framework serialisation or reflection must not create a second instance; a
  plain holder singleton is breakable, and an `enum` is the answer there.
- Don't choose when: tests must reset the singleton between runs; a holder has no reset.

## Interview angle
- Probed as "how does the holder idiom stay thread-safe without `synchronized`?"
- Common wrong answer: "the holder idiom has no downsides"; a constructor that throws leaves the
  class dead until restart.
- Strong answer: the JVM initialises `Holder` once under its initialisation lock, other threads
  wait and then see the finished instance, and laziness comes from `Holder` being a separate class;
  then name the failure mode, and end with the senior turn: in Spring, use a singleton bean.

## Related
- [[Java core MOC]]: class initialisation is something the JVM does for you, and this idiom is the
  usual way to borrow it for a lazy singleton.
- [[Concurrency MOC]]: it is the thread-safe singleton that needs no explicit lock in the code.
- [[Since JDK 5, double-checked locking with non-final fields needs a volatile field so the lock-free first read sees the constructor's writes]]:
  the alternative that does its own locking and retries after a failed constructor, which the
  holder cannot.
- [[A Spring singleton bean is one instance per container and bean definition, not one per class loader as in the GoF pattern]]:
  what replaces a hand-written singleton inside a Spring application.
- Written up in win-interview: backend/java/docs/java-memory-model.md, section 2.6.4
