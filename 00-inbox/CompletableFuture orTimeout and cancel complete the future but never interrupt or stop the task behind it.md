---
tags: [java, concurrency, completablefuture, timeout, interview]
status: draft
author: claude
up: ["[[Concurrency MOC]]"]
source: "https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/CompletableFuture.html"
created: 2026-10-07
review: unjudged
---
# CompletableFuture orTimeout and cancel complete the future but never interrupt or stop the task behind it

## Core idea
A `CompletableFuture` has no direct control over the computation that completes it. `orTimeout`
(Java 9) completes the future with a `TimeoutException` if nothing else completed it first, and
`completeOnTimeout` completes it with a fallback value. `cancel(true)` completes it with a
`CancellationException`, and its interrupt flag has no effect, because the implementation does not
use interrupts. In the tests the caller gets the `TimeoutException` or `CancellationException`
while the task keeps its thread, is never interrupted, and its late answer is thrown away. To stop
the work itself, put a timeout on the call, such as the HTTP client's read timeout or a JDBC query
timeout.

## Why choose / why not
- Use `orTimeout` or `completeOnTimeout` when: the caller must stop waiting after a deadline and
  can answer with an error or a fallback value.
- Don't treat it as cancellation when: the task holds a thread. It keeps running
  after the callers gave up, and retries pile up on the same pool; set the deadline inside the
  call instead.
- Use `ExecutorService.invokeAll(tasks, timeout, unit)` when: a batch shares one deadline and the
  tasks react to interrupts; unfinished tasks are cancelled on return, but cancelling only
  interrupts.
- Keep callbacks after `orTimeout` trivial on JDK 21 to 24: the timeout fires on one shared daemon
  thread, so a plain `thenApply` usually runs there (read in the source, not tested).

## Interview angle
- Probed as "how do you handle errors and timeouts in a `CompletableFuture` chain?"
- Common wrong answers: "`orTimeout` cancels the call", and "`cancel(true)` interrupts the task."
- Strong answer: both only complete the future, and the work keeps running, visible as busy
  threads long after the callers gave up; the real deadline lives in the I/O call, and an
  interrupt stops a task only if it checks for it. A `FutureTask` returned by `submit` differs:
  its `cancel(true)` does interrupt the running thread.

## Related
- [[Concurrency MOC]]: the timeout and cancellation questions that follow any async design.
- [[Parallel streams share the JVM-wide common ForkJoinPool]]: async steps without an executor run
  in that same pool, so a call that outlives its timeout keeps holding a worker every other user
  of the pool needs.
- [[An HTTP client's read timeout is measured between reads, so it does not bound the whole call]]:
  the per-call timeout recommended here has its own limit, which that note explains.
- [[A @Transactional timeout fails the next database access after the deadline instead of interrupting the method]]:
  another deadline that does not interrupt the running code.
- Written up in win-interview: backend/java/docs/thread-pools-and-async.md, section 2.6 (Timeouts and cancellation)
