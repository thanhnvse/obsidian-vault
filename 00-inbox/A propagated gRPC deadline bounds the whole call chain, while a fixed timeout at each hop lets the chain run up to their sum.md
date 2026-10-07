---
tags: [grpc, microservices, timeouts, resilience, interview]
status: draft
author: claude
up: ["[[Microservices and messaging MOC]]"]
source: "https://grpc.io/docs/guides/deadlines/"
created: 2026-10-07
review: unjudged
---
# A propagated gRPC deadline bounds the whole call chain, while a fixed timeout at each hop lets the chain run up to their sum

## Core idea
A gRPC deadline is a point in time the call must not go past, and by default none is set, so a client
can wait effectively forever. The client sends the time left as the `grpc-timeout` header; when the
deadline passes the client fails with `DEADLINE_EXCEEDED` and the server's call is cancelled. Where
the implementation propagates deadlines (the documentation says some do), a call a service makes
while handling an incoming one carries the time left, with the elapsed time already deducted. In the
write-up's grpc-java tests A called B with no deadline of its own and B still saw the caller's. With
300 ms at the edge the chain ends at 300 ms, while a fixed 300 ms timeout at each of three hops
allows up to 900 ms (arithmetic), long after the first caller gave up. Propagation lives in the call
context, which is bound to the thread: a downstream call from another thread carried no deadline
until the work was wrapped in the handler's `Context`.

## Why choose / why not
- Set a deadline at the edge always: the gRPC documentation says to set an explicit, realistic
  deadline in clients.
- Prefer a propagated deadline over per-hop timeouts when: services call services; one budget bounds
  the chain, and cancellation stops orphan (mồ côi) work, where B keeps computing for a client that
  has left.
- Wrap a hand-off to another thread in the handler's context (`Context.current().wrap(...)`) when:
  a handler uses a pool; otherwise the deadline is lost. Tracing contexts need the same care, which
  the write-up did not test.
- Don't expect a budget from a plain REST chain: no built-in equivalent was found, so design a header
  with the time left that every service honours.
- Don't start a retry that cannot finish before the deadline: the deadline is the upper bound for
  every retry.

## Interview angle
- Probed as "the edge call has a 2 s deadline and the third service takes 5 s: what happens?"
- Common wrong answers: "a 1-second timeout at each of three hops means at most 1 second" (it is up
  to 3), or "gRPC always propagates the deadline".
- Strong answer: the client gets `DEADLINE_EXCEEDED` at 2 s, the stream is cancelled, and each
  downstream call carrying the remaining time is cancelled too; a hop that called from another thread
  without the context leaves the third service working; with no deadlines nothing stops.

## Related
- [[Microservices and messaging MOC]]: the map for slow callees; a deadline is the budget side of
  that problem.
- [[An HTTP client's read timeout is measured between reads, so it does not bound the whole call]]:
  the REST side of the same lesson; a per-read timeout does not bound a call, a whole-call deadline
  does.
- [[A synchronous service call couples the caller to the callee's availability and latency]]: in a
  chain each wait adds to the latency; a propagated deadline caps the sum.
- [[Clients that retry a struggling service immediately and without limit keep it from recovering]]:
  retries must be bounded, and the deadline is their upper bound.
- Written up in win-interview: backend/docs/microservice-infrastructure.md, section 2.7
