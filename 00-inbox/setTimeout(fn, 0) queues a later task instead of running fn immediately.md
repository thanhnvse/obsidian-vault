---
tags: [javascript, frontend, event-loop, timers, interview]
status: draft
author: claude
up: ["[[Frontend interview MOC]]"]
source: "https://html.spec.whatwg.org/multipage/timers-and-user-prompts.html#timer-initialisation-steps"
created: 2026-10-01
score: 0.861
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# setTimeout(fn, 0) queues a later task instead of running fn immediately

## Core idea
`setTimeout` runs the HTML timer initialisation steps: they wait at least the timeout and then
queue a task on the timer task source that calls the callback. With a timeout of 0, `fn` is never
called during the task that scheduled it: that task finishes, the microtask checkpoint after it
empties the microtask queue, and only then can the event loop pick the timer task, possibly after
other tasks. The timeout is also only a lower bound. When `setTimeout` is called from a timer task
whose timer nesting level is greater than 5, a timeout below 4 ms is raised to 4 ms, and the
standard lets a user agent wait a further implementation-defined time.

## Why choose / why not
- Use `setTimeout(fn, 0)` when: code must run after the current task and its microtasks, with the
  browser free to handle other tasks first, such as yielding inside a long loop; it works in every
  browser and in Node.
- Don't use it for a long chain of small chunks: past five levels of nesting each hop costs at
  least 4 ms, so prefer `scheduler.yield()` where it exists.
- Don't use it when the work must run before the next task or frame: use `queueMicrotask`, or
  `requestAnimationFrame` for DOM writes that belong in the next frame.
- Don't use it as a precise timer: the standard allows extra delay, and browsers throttle timers
  in background tabs.

## Interview angle
- Probed as "why does `setTimeout(fn, 0)` run after `Promise.resolve().then(...)` even though it
  was scheduled first?"
- Common wrong answer: "0 ms means it runs immediately", or "timers and promise callbacks run in
  the order they were scheduled".
- Strong answer: `setTimeout` queues a task; the current task and every queued microtask run
  first; the delay is a minimum, clamped to 4 ms for deeply nested timers.

## Related
- [[The microtask queue is drained completely after every task, before the next task or a render]]:
  it explains why the promise callback runs before the timer.
- [[Awaiting an already-resolved promise does not let the browser render or handle input]]: the
  case where this task boundary is the fix, because `await` alone does not create one.
