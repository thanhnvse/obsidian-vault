---
tags: [java, spring, http-client, resilience, interview]
status: draft
author: claude
up: ["[[Spring MOC]]"]
source: "https://docs.spring.io/spring-framework/docs/6.1.x/javadoc-api/org/springframework/web/client/ResourceAccessException.html"
created: 2026-10-01
score: 0.89
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A Spring ResourceAccessException means no response arrived, so the request's outcome is unknown

## Core idea
Spring's synchronous clients, `RestTemplate` and `RestClient`, throw `ResourceAccessException` when
an I/O error occurs before a response arrives, such as a refused connection or a connect or read
timeout. After a read timeout the server may already have received and processed the request, so
the client cannot tell whether the request took effect. An error status is different: the server
did answer, and the clients report it as a `RestClientResponseException`, by default an
`HttpClientErrorException` for 4xx or an `HttpServerErrorException` for 5xx. Both exception types
extend `RestClientException`, so a single `catch (RestClientException e)` treats an unknown outcome
like an answered failure and can retry a non-idempotent payment request that may already have
succeeded. A lab on Spring Framework 6.1.2 confirmed a `ResourceAccessException` caused by
`SocketTimeoutException` or `HttpTimeoutException` on a read timeout, and an
`HttpServerErrorException` on a 500.

## Why choose / why not
- Retry a `ResourceAccessException` only when: the request is idempotent, or carries an idempotency
  key that the server deduplicates; the first attempt may have succeeded.
- Catch `ResourceAccessException` separately from `RestClientResponseException` when: the caller
  retries, alerts or compensates; an unknown outcome and an answered failure need different
  decisions.
- Don't resend a non-idempotent request after a timeout without first checking its outcome, for
  example by querying the server with a request ID the client generated.

## Interview angle
- Probed as "a call to the payment service timed out. Do you retry it?".
- Common wrong answer: "yes, retry any exception from the client a few times."
- Strong answer: a timeout is "no answer", not "an error answer", so the outcome is unknown; retry
  only an idempotent or deduplicated request, or check the outcome first.

## Related
- [[An HTTP client's read timeout is measured between reads, so it does not bound the whole call]]:
  the read timeout is the usual cause of a `ResourceAccessException`, and that note says what it
  measures.
- [[Committing Kafka offsets after processing gives at-least-once delivery, so consumers must be idempotent]]:
  the same principle on the messaging side; once a request can be repeated after an unknown outcome,
  the receiver has to be idempotent.
