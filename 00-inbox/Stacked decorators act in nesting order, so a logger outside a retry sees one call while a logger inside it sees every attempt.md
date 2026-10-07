---
tags: [java, design-patterns, decorator, proxy, interview]
status: draft
author: claude
up: ["[[Java core MOC]]", "[[Spring MOC]]"]
source: ""
created: 2026-10-07
review: unjudged
---
# Stacked decorators act in nesting order, so a logger outside a retry sees one call while a logger inside it sees every attempt

## Core idea
A decorator wraps an object that has the same interface and adds behaviour, and wrappers can be
stacked, so the outermost one sees the whole call. In the lab a price source fails twice and a retry
wrapper tries up to three times. With the logger outside the retry the log has one line; with the
logger inside it, three lines, one per attempt. Same classes, different behaviour. Spring builds the
same nesting for aspects: the aspect with the lower `@Order` value is outermost, which is why
transaction and cache advice need an explicit order; with the transaction outermost, the cache
eviction runs before the commit and a concurrent reader can reload the old row. A proxy has the same
shape but controls access to its target.

## Why choose / why not
- Stack decorators when: independent behaviours such as logging, retry or buffering must be added to
  one interface without editing the class, and the client can choose the order.
- State the order in one place and test it when: the order changes behaviour, as with log and retry
  or transaction and cache; use one configuration class and a recording log in the test, as the lab
  does.
- Don't stack many: each wrapper adds a frame to every stack trace and one more place where
  behaviour hides; five decorators make "what actually runs" a question for a debugger.
- Expect the concrete type to disappear: `Collections.unmodifiableList(arrayList)` is a `List`, not
  an `ArrayList`, so a cast fails and the methods outside the interface are gone.

## Interview angle
- Probed as "what is the difference between a decorator and a proxy?"
- Common wrong answer: "they are the same thing"; the shape is the same, the intent is not.
- Strong answer: a decorator adds behaviour and the client composes it around a target it hands in;
  a proxy controls access, such as lazy creation, a permission check or interception, and a
  framework usually inserts it. Then give the runtime consequence, the order, with the log-and-retry
  example. In Spring, `@Transactional` is a proxy and a `@Primary` logging wrapper bean is a decorator.

## Related
- [[Java core MOC]]: the JDK has decorators, such as `BufferedInputStream` over an
  `InputStream`, so the pattern is a way to read its classes.
- [[Spring MOC]]: a Spring bean with advice is a proxy around one target, with the advice as a chain
  of wrappers whose order the container decides.
- [[Spring AOP applies @Aspect annotations through proxies, without the AspectJ weaver]]: the
  aspects whose `@Order` decides the nesting run as advice inside that proxy.
- Written up in win-interview: backend/java/docs/design-patterns.md, section 9
