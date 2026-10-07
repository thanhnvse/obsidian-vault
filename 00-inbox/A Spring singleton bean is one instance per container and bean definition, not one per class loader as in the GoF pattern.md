---
tags: [java, spring, singleton, design-patterns, interview]
status: draft
author: claude
up: ["[[Spring MOC]]", "[[Java core MOC]]"]
source: "https://docs.spring.io/spring-framework/reference/core/beans/factory-scopes.html#beans-factory-scopes-singleton"
created: 2026-10-07
review: unjudged
---
# A Spring singleton bean is one instance per container and bean definition, not one per class loader as in the GoF pattern

## Core idea
The GoF singleton hard-codes its scope in the class (a private constructor and a static field), so
there is one instance per class loader. A Spring singleton is enforced by the container's registry,
and the reference describes its scope as per-container and per-bean. A lab on Spring Framework
6.1.2 shows what follows: a singleton bean is the same instance on every lookup in one container;
two bean definitions of one class give two instances; two containers in one class loader each hold
their own; a prototype bean is created again on every lookup. The container also makes creation
thread-safe, but it does nothing for the bean's mutable fields afterwards, and one instance serves
every request thread, so singleton beans should be stateless.

## Why choose / why not
- Choose a singleton bean, injected through the constructor, when: a Spring or other DI application
  needs one shared collaborator; the dependency is visible in the constructor, a test can pass a
  fake, and the bean gets `@PostConstruct`, `@PreDestroy` and ordered shutdown.
- Choose a hand-written singleton, the holder idiom or an `enum`, when: it is plain Java with no
  container.
- Don't read "singleton" as one per JVM: two `@Bean` methods of the same class, or two containers,
  give two instances.
- Don't keep mutable state in a singleton bean: when it must hold state, use an atomic class, a
  lock, or a scope that is not shared.

## Interview angle
- Probed as "Spring singleton vs the singleton pattern?"
- Common wrong answer: "Spring beans are singletons, so Spring uses the singleton pattern."
- Strong answer: per container and per bean definition, not per class loader; the container enforces
  it, so the class can have a public constructor and callers can be tested with fakes; creation is
  thread-safe but the bean's mutable state is not, so keep singleton beans stateless.

## Related
- [[Spring MOC]]: singleton is the default scope of every bean the container manages.
- [[Java core MOC]]: the GoF half, one instance per class loader, is JVM behaviour: each class
  loader defines its own copy of the class and its own static field.
- [[The holder idiom stays thread-safe without synchronized because the JVM initialises the Holder class once, under its class-initialisation lock]]:
  the hand-written alternative for plain Java.
- [[Since JDK 5, double-checked locking with non-final fields needs a volatile field so the lock-free first read sees the constructor's writes]]:
  in spring-beans 6.1.2 the registry creates a missing singleton with the same shape, a lookup in a
  `ConcurrentHashMap` and a second check inside `synchronized`.
- [[Spring does not call @PreDestroy on prototype-scoped beans]]: the prototype scope contrasted
  above; the container creates one per lookup and keeps no record of it.
- Written up in win-interview: backend/java/docs/design-patterns.md, section 3 (also java-memory-model.md, section 3.3)
