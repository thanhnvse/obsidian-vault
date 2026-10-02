---
tags: [javascript, frontend, event-loop, interview]
status: draft
author: claude
up: ["[[Frontend interview MOC]]"]
source: "https://html.spec.whatwg.org/multipage/webappapis.html#event-loop-processing-model"
created: 2026-10-01
score: 0.897
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# The microtask queue is drained completely after every task, before the next task or a render

## Core idea
Each turn of the HTML event loop takes one task from a task queue, runs it, and then performs a
microtask checkpoint. The checkpoint keeps dequeuing and running microtasks while the microtask
queue is not empty, so a microtask queued by another microtask runs in the same checkpoint.
Promise reactions are microtasks because HTML implements the ECMAScript `HostEnqueuePromiseJob`
hook by queuing a microtask, and `queueMicrotask()` queues one directly. Updating the rendering is
itself queued as a task on the rendering task source, so everything queued as a microtask during a
task runs before the next task and before the next render. Code that keeps queuing microtasks
never lets a timer, an event handler or a frame run.

## Why choose / why not
- Use a microtask (`queueMicrotask`, a promise callback) when: work must run after the current
  code returns but before any event handler, timer or render sees the state, such as flushing
  several synchronous updates as one batch.
- Don't use a microtask when: the goal is to let the page respond. It runs before input and
  rendering, so it cannot break up a long task; yield a task instead.
- Don't let a microtask queue another microtask without a bound: the checkpoint ends only when the
  queue is empty, so recursive microtasks freeze the page like an infinite loop.

## Interview angle
- Probed with an ordering puzzle: a timer callback that queues a promise callback, with a second
  timer queued behind it. The promise callback runs before the second timer.
- Common wrong answer: "the loop runs all ready tasks, then all microtasks", or "tasks and
  microtasks run in the order they were scheduled".
- Strong answer: one task, then the whole microtask queue including microtasks added meanwhile,
  then maybe a render. Name `Promise.then`, `await` continuations and `queueMicrotask` as
  microtasks, and timers and DOM events as tasks.

## Related
- [[Awaiting an already-resolved promise does not let the browser render or handle input]]: the
  trap this rule sets for async loops, because their `await` only reaches the microtask queue.
- [[setTimeout(fn, 0) queues a later task instead of running fn immediately]]: the task side of
  the same rule, and why a 0 ms timer waits for every queued microtask.
