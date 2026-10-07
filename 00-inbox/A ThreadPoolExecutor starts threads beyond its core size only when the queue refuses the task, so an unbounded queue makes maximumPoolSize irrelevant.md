---
tags: [java, concurrency, thread-pool, executor, interview]
status: draft
author: claude
up: ["[[Concurrency MOC]]"]
source: "https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ThreadPoolExecutor.html"
created: 2026-10-07
review: unjudged
---
# A ThreadPoolExecutor starts threads beyond its core size only when the queue refuses the task, so an unbounded queue makes maximumPoolSize irrelevant

## Core idea
For each new task, `ThreadPoolExecutor.execute` applies three steps in order. Below `corePoolSize`
it starts a thread, even if other workers are idle. Otherwise it offers the task to the queue. Only
if the queue refuses does it start a thread up to `maximumPoolSize`; at the maximum the rejection
handler decides. With core 0, a re-check after the queue accepts also starts one worker when none
is running. With core 1, maximum 3 and a queue of 2, tasks 2 and 3 wait in the queue, tasks 4
and 5 start two more threads, and task 6 is rejected. So the queue type, not the maximum, decides
the behaviour under load. `newFixedThreadPool` has an unbounded `LinkedBlockingQueue` and turns
overload into queue growth; `newCachedThreadPool` has a `SynchronousQueue` and no thread limit, and
turns overload into one thread per concurrent task. Neither rejects a task because of load.

## Why choose / why not
- Build a `ThreadPoolExecutor` by hand when: the pool is production code: a bounded queue such as
  `ArrayBlockingQueue`, threads named after the work, and a rejection policy chosen on purpose.
- Don't choose `newFixedThreadPool` when: a dependency can be slow, because overload then becomes
  latency and memory, never an error. `newCachedThreadPool` fails the other way, with thousands of
  threads → use a bounded pool, or virtual threads for blocking work.
- Pick the rejection policy by who submits: `AbortPolicy` when the caller can turn "busy" into a
  429 or a retry; `CallerRunsPolicy` for back-pressure on a producer that can slow down, never on a
  request thread whose own latency matters; `Discard*` only when nobody waits, because the future
  of a discarded `submit` never completes.
- Size the pool together with the connection pool behind it: 80 threads in front of 10
  connections means 70 threads waiting for a connection.

## Interview angle
- Probed as "explain the `ThreadPoolExecutor` parameters; when does it create a thread beyond
  core?"
- Common wrong answer: "the pool grows to `maximumPoolSize` when all core threads are busy."
- Strong answer: core, then queue, then extra threads only when the queue refuses, then the
  handler. Derive that with an unbounded queue the maximum never matters (core 0 with an unbounded
  queue runs one thread), and that the extra thread runs the newest task, so a new task overtakes
  the queued ones.

## Related
- [[Concurrency MOC]]: its question applied to tasks instead of data: what happens when many tasks
  arrive at once, and which limit fails first (threads, queue or connections).
- [[A small database connection pool often beats a large one, because extra connections only time-slice the same cores]]:
  threads that talk to a database can do no more than its connection pool allows, so the two are
  sized together.
- [[CompletableFuture orTimeout and cancel complete the future but never interrupt or stop the task behind it]]:
  a timed-out call keeps its pool thread busy after the caller gave up, so a pool can stay full
  although nobody is waiting.
- Written up in win-interview: backend/java/docs/thread-pools-and-async.md, sections 2.1 to 2.3
