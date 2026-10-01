---
tags: [moc, frontend, interview]
type: moc
status: draft
author: claude
created: 2026-10-01
---
# Frontend interview MOC

Entry point for the frontend and full-stack interview topics: what does the browser actually run
next, and what does the user see while it runs? Long-form write-ups with runnable proof live in
the `win-interview` repository; these notes are the atomic claims to rewrite in your own words.

Sources cite the WHATWG HTML Living Standard and the ECMAScript draft as published on
2026-10-01. Living standards change; check a claim against the current text before relying on it.

## Event loop
- [[The microtask queue is drained completely after every task, before the next task or a render]]: the core rule that answers almost every "what runs first?" puzzle.
- [[An async function runs synchronously until its first await]]: explains where code inside an `async` function lands among the synchronous logs.
- [[Awaiting an already-resolved promise does not let the browser render or handle input]]: the trap behind "my async loop still freezes the page".
- [[setTimeout(fn, 0) queues a later task instead of running fn immediately]]: the task-side answer to "why does a 0 ms timer run after a promise callback?".

## Open questions
- Not yet captured: `scheduler.yield()` versus `setTimeout` for breaking up long tasks, `requestAnimationFrame` timing, and Node's `process.nextTick` and `setImmediate` queues.
