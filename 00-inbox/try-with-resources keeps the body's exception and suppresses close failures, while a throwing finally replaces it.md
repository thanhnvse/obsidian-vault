---
tags: [java, java-core, exceptions, try-with-resources, interview]
status: draft
author: claude
up: ["[[Java core MOC]]"]
source: "https://docs.oracle.com/javase/specs/jls/se21/html/jls-14.html#jls-14.20.3"
created: 2026-10-07
review: unjudged
---
# try-with-resources keeps the body's exception and suppresses close failures, while a throwing finally replaces it

## Core idea
If a `finally` block completes abruptly (`return`, `throw` or `break`), the `try` statement
completes for that same reason and whatever the `try` or `catch` was doing is discarded
(JLS 14.20.2). `try { throw A; } finally { throw B; }` throws B only: A is neither the cause nor
suppressed. With `finally { return v; }` the call returns normally and A is gone. So a hand-written
`finally { r.close(); }` lets a failing `close()` hide the failure of the body. try-with-resources
compiles to the nested `try`/`finally` with suppression built in (JLS 14.20.3): resources close in
reverse order, and when the body threw A and `close()` threw B, A is thrown with B in
`A.getSuppressed()`. When the body is fine, the first close failure is thrown and later ones are
suppressed on it.

## Why choose / why not
- Use try-with-resources when: the thing is an `AutoCloseable` such as a connection, a statement, a
  `Files.lines` stream, a `Scanner` or an HTTP response; the body's exception wins and every
  resource is still closed when an earlier `close()` throws.
- Hand-write `finally` when: the cleanup is not an `AutoCloseable`; catch the cleanup failure there
  and attach it with `addSuppressed`, or the original exception is lost.
- Never `return` or `throw` from `finally`: it silently replaces the failure in flight.
- Don't count on `finally` when: the JVM exits; `System.exit(3)` inside the `try` skips it.

## Interview angle
- Asked as "what does `finally` do, and what can go wrong?" or "what are suppressed exceptions?".
- Common wrong answer: "`finally` always runs", which ignores `System.exit` and what `finally`
  does when it throws.
- Strong answer: the return value is fixed before `finally` runs; a `return` in it discards an
  exception and a throw replaces it with nothing attached; try-with-resources keeps the body's
  exception and lists close failures as suppressed, in closing order.

## Related
- [[Object.finalize is deprecated for removal, so cleanup belongs in close() with a Cleaner as the safety net]]:
  that note says cleanup belongs in an `AutoCloseable`'s `close()` used with try-with-resources;
  this one covers what happens when `close()` itself throws.
- [[Java core MOC]]: the map entry for how exceptions behave when cleanup code fails.
- Written up in win-interview: backend/java/docs/exceptions.md, sections 2.3 and 2.4
