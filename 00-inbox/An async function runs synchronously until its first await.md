---
tags: [javascript, frontend, event-loop, async, interview]
status: draft
author: claude
up: ["[[Frontend interview MOC]]"]
source: "https://tc39.es/ecma262/multipage/control-abstraction-objects.html#sec-async-functions-abstract-operations-async-function-start"
created: 2026-10-01
score: 0.91
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# An async function runs synchronously until its first await

## Core idea
Calling an async function creates the promise it will return and then starts evaluating its body
at once, on the caller's stack (ECMAScript `EvaluateAsyncFunctionBody` calls `AsyncFunctionStart`).
The code before the first `await` therefore runs synchronously, in line with the caller's code. At
the first `await` the function suspends and the caller continues with the pending promise; the
code after the `await` resumes later in a promise reaction job, which HTML runs as a microtask. An
async function that never awaits runs its whole body synchronously. An exception thrown before the
first `await` does not reach the caller as a throw: it rejects the returned promise.

## Why choose / why not
- Rely on it when: setup must happen right away, for example registering a listener or copying
  arguments before the caller changes them, so that work goes before the first `await`.
- Don't rely on `async` to defer work: the keyword does not move the body off the current task,
  so CPU work before the first `await` blocks the caller exactly like a plain function.
- Don't expect a synchronous error: a `try/catch` around a call that is not awaited misses an
  exception thrown before the first `await`; await the call or attach `.catch`.

## Interview angle
- Probed with a puzzle where an async IIFE logs before its `await`, and the question is where that
  log lands among the surrounding synchronous logs.
- Common wrong answer: "everything inside an async function runs later, after the synchronous
  code".
- Strong answer: the body runs synchronously up to the first `await`; only the continuation is a
  microtask, queued in order with the other promise callbacks.

## Related
- [[The microtask queue is drained completely after every task, before the next task or a render]]:
  it says when the continuation after the first `await` actually runs.
- [[Awaiting an already-resolved promise does not let the browser render or handle input]]: the
  same suspension point seen from the page, where it yields to the caller and the microtask queue
  but not to the browser.
