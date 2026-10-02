---
tags: [javascript, frontend, event-loop, async, performance, interview]
status: draft
author: claude
up: ["[[Frontend interview MOC]]"]
source: "https://tc39.es/ecma262/multipage/control-abstraction-objects.html#await"
created: 2026-10-01
score: 0.9
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Awaiting an already-resolved promise does not let the browser render or handle input

## Core idea
ECMAScript `Await` converts its operand with `PromiseResolve` and attaches the continuation with
`PerformPromiseThen`. When that promise is already fulfilled, `PerformPromiseThen` enqueues the
reaction job immediately through `HostEnqueuePromiseJob`, which HTML implements by queuing a
microtask. The async function is suspended only until that microtask runs, and the event loop
drains the microtask queue before it takes the next task. Click events and rendering updates are
queued as tasks, so a loop that awaits `Promise.resolve()` or a plain value between chunks of work
still blocks input and rendering until the whole loop finishes. Awaiting a promise that a
`setTimeout` callback resolves does create a task boundary, so the event loop can run other tasks
and render before the function continues.

## Why choose / why not
- An `await` on a resolved value is fine when: the goal is ordering, running after the current
  synchronous code, or a uniform async API; it costs one microtask, not a frame.
- Don't use it to keep the page responsive: the loop stays one long task. Yield a task between
  chunks instead, with `scheduler.yield()` where it exists and `setTimeout` as the fallback.
- Move the work to a Web Worker when: each chunk is CPU-bound and still too slow after chunking,
  and it does not need the DOM.

## Interview angle
- Probed as "does `await` let the browser render or handle a click?", or as a review of an `async`
  loop that is meant not to freeze the page.
- Common wrong answer: "yes, `await` yields to the event loop, so the UI stays responsive".
- Strong answer: `await` yields only to the microtask queue, which is drained before the next
  task; responsiveness needs a task boundary or a worker.

## Related
- [[The microtask queue is drained completely after every task, before the next task or a render]]:
  the general rule; this note differs by showing what it means for `await` inside a long loop.
- [[setTimeout(fn, 0) queues a later task instead of running fn immediately]]: the usual fix,
  because it creates the task boundary that `await` alone does not.
- [[An async function runs synchronously until its first await]]: the other half of the same
  misconception, that `async` code automatically runs later.
- [[asyncio suits I-O-bound work, not CPU-bound work]]: the same single-thread rule in Python,
  where a coroutine that does not give up control blocks every other coroutine on the loop.
