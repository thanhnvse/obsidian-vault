---
tags: [java, spring, logging, observability, interview]
status: draft
author: claude
up: ["[[Spring MOC]]", "[[Ops and cloud MOC]]"]
source: "https://logback.qos.ch/manual/mdc.html"
created: 2026-10-07
review: unjudged
---
# The MDC stays on the thread that set it, so a task run on another thread logs without the caller's request id by default

## Core idea
The MDC (Mapped Diagnostic Context) is a per-thread map that the logging framework copies into
every event logged on that thread, so a filter can set `requestId` once and every line of that
request carries it. Because it is a `ThreadLocal`, the map does not follow work to another thread.
In a lab (Logback 1.4.14, Spring Boot 3.2.1) a new thread, a pool task, a virtual thread and an
`@Async` method on an executor without a decorator all logged with no `requestId`. A
`TaskDecorator` fixes it: copy `MDC.getCopyOfContextMap()` on the submitting thread, set it on the worker, and restore
the worker's previous map in a `finally` block. Skip the restore and a pooled thread keeps the
last task's id, so a later request logs a stale (cũ, lỗi thời) id from an earlier one.

## Why choose / why not
- Decorate the executor with a `TaskDecorator` when: pooled or `@Async` work must log with the
  request id. Spring Framework 6.1.2 ships `ContextPropagatingTaskDecorator`, but it carries
  nothing until a `ThreadLocalAccessor` is registered for the MDC key.
- Pass the few ids as parameters when: the code is not on the request path; decorating every
  executor costs a captured map per task and a stale-id bug if the restore is forgotten.
- Use the Reactor `Context` when: the chain is reactive; an operator after `publishOn` runs on
  another thread and sees no MDC. Register an MDC accessor and turn on automatic context
  propagation (`spring.reactor.context-propagation=auto`; the default is `limited`) to have the
  `ThreadLocal`s restored on each hop.

## Interview angle
- Probed as "my log lines lose the request id inside `@Async` code; why, and what do you do?"
- Common wrong answers: "the MDC is global, so the id is on every line", and "set the MDC in the
  task and we are done", which leaves the next task on that pool thread with the old id.
- Strong answer: the MDC is a `ThreadLocal`; a `TaskDecorator` copies it on submit, sets it on the
  worker and restores the previous map in `finally`; reactive code carries it in the Reactor
  `Context` with automatic propagation.

## Related
- [[Work handed to another thread runs outside the caller's Spring transaction]]: the same
  thread-bound mechanism; the transaction, like the MDC, stays on the calling thread, so async work
  loses both.
- [[A servlet filter sees every request, an interceptor only requests that found a handler, and an aspect only method calls on a bean]]:
  the filter is where the correlation id and MDC are set, because it sees every request.
- [[Spring MOC]]: the `TaskDecorator` is wired onto the Spring-managed executor behind `@Async`.
- [[Ops and cloud MOC]]: a request id on every log line is how an operator follows one request
  through an incident.
- Written up in win-interview: backend/java/docs/observability.md, section 2.2
