---
tags: [java, java-core, exceptions, logging, interview]
status: draft
author: claude
up: ["[[Java core MOC]]"]
source: "https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Throwable.html"
created: 2026-10-07
review: unjudged
---
# Wrapping an exception with only its getMessage() loses the cause, the original type and the frame where it failed

## Core idea
`new X("failed: " + e.getMessage())` keeps the text of the original failure and nothing else: the
cause, the `Caused by:` line, the original type (so no `instanceof` on it) and the frame where it
happened are gone. `new X("msg", cause)` keeps them: `getCause()` returns the cause, and the printed
trace shows `Caused by:` with the failing method. The wrapper's own trace starts at the line that
built it, the translation site, while the cause's trace starts where the failure was created.
Logging has the same trap: `log.error("could not save order {}", id, e)` records the exception with
its stack trace and causes, while `log.error("could not save: " + e.getMessage())` records one line
of text with no stack trace at all.

## Why choose / why not
- Pass the cause when: an adapter translates a low-level exception, for example a `SQLException`
  with SQL state `23505` into `OrderAlreadyExists(orderId, e)`; the service sees the domain type and
  the driver exception stays one `getCause()` away.
- Translate once per layer boundary when: wrapping; each wrapper captures its own stack trace, so
  wrapping at every call builds one trace per wrapper.
- Let your own exception family through when: a blanket `catch (RuntimeException e)` surrounds calls
  that already throw `OrderNotFound`; rethrow with `catch (DomainException e) { throw e; }` first,
  or a handler that matches by type sees `IllegalStateException` and a 404 becomes a 500.
- Log once, at the top, when: you handle it, with the exception object as the last argument; layers
  that only translate or pass it on do not log, or one failure becomes N log events.

## Interview angle
- Asked as "how do you not lose the root cause?" or "what happens to the stack trace when you wrap
  an exception?".
- Common wrong answer: "I catch `Exception` and log `e.getMessage()`", or "we wrap everything in
  `RuntimeException` to keep it simple".
- Strong answer: pass the cause; the wrapper gets its own trace from the line that built it and the
  original stays in `getCause()` with its own; translate once per boundary and log once with the
  exception object.

## Related
- [[A Spring ResourceAccessException means no response arrived, so the request's outcome is unknown]]:
  the exception type is what callers act on (retry or not), and a message-only copy throws that
  type away.
- [[A checked exception commits a Spring @Transactional method by default]]: the write-up's other
  reason for unchecked domain exceptions in services; read it before choosing the wrapper's type.
- [[Java core MOC]]: the map entry for how exceptions carry information through layers.
- Written up in win-interview: backend/java/docs/exceptions.md, sections 2.5 and 3.3
