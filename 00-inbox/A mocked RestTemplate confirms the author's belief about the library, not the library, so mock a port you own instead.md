---
tags: [java, spring, testing, mockito, interview]
status: draft
author: claude
up: ["[[Spring MOC]]"]
source: "https://growing-object-oriented-software.com/"
created: 2026-10-07
review: unjudged
---
# A mocked RestTemplate confirms the author's belief about the library, not the library, so mock a port you own instead

## Core idea
Freeman and Pryce's rule is to mock types you own, not third-party ones, because a mock of a library
type holds your belief (giả định) about it, and the test stays green when the belief is wrong. In a
lab, a client's test stubbed `RestTemplate` to return `null` for "a 404" and passed, while the real
`RestTemplate` throws `HttpClientErrorException.NotFound`, so production did the opposite. The
remedy has two halves: define a port in your own words, such as `RateSource.rateFor(pair)` returning
`Optional`, and mock that in use-case tests; then write one adapter over the library and test it
against real HTTP. The lab's adapter test gave `Optional.empty()` on a 404 and
`HttpServerErrorException` on a 500, so a server error is not hidden as an unknown pair.

## Why choose / why not
- Mock your own port when: testing a use case at an I/O boundary; the test stays plain and fast. A
  port at a real I/O boundary is the one interface allowed before it has two call sites.
- Test the adapter against the real thing, a real HTTP server or database, when: it wraps a library;
  that test is where your belief about the library is checked.
- Don't mock value objects and records (build them), the class under test (a spy on the subject
  means the test reaches inside it), or `RestTemplate`, `JdbcTemplate` and `EntityManager` (the test
  then checks the mock, not the query).
- Keep the adapter thin: the cost is one extra interface and one test with a real server.

## Interview angle
- Probed as "why not mock `RestTemplate`?".
- Common wrong answer: "mock everything so the unit test is isolated."
- Strong answer: you do not own `RestTemplate`, so the mock holds your belief about it; wrap it in a
  port, test the adapter against a real server, mock the port; the 404 that returns `null` in the
  mock and throws in the real client is the example.

## Related
- [[Spring MOC]]: the parent map; its HTTP clients section covers the libraries an adapter wraps.
- [[A Spring ResourceAccessException means no response arrived, so the request's outcome is unknown]]:
  the real client separates no answer (`ResourceAccessException`) from an error status, which is the
  behaviour a mocked `RestTemplate` replaces with a belief; the lab's adapter test pins the status
  cases (404, 500), and that note covers the no-answer case.

Written up in win-interview: backend/java/docs/testing-strategy.md, sections 1 and 2.2
