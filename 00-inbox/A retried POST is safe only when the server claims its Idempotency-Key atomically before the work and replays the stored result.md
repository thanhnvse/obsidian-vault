---
tags: [rest, api-design, idempotency, reliability, interview]
status: draft
author: claude
up: ["[[Microservices and messaging MOC]]"]
source: "https://datatracker.ietf.org/doc/html/draft-ietf-httpapi-idempotency-key-header-07"
created: 2026-10-07
review: unjudged
---
# A retried POST is safe only when the server claims its Idempotency-Key atomically before the work and replays the stored result

## Core idea
A client that times out on `POST /payments` cannot tell a lost request from a lost response, and
HTTP says not to retry a `POST` blindly. The `Idempotency-Key` header gives it a way to know. It comes
from an IETF Internet-Draft (draft-07), not an RFC; its datatracker state on 2026-10-06 was Expired.
The client sends the same unique key on every attempt. A retry after the original finished gets the
stored result, success or error; a retry while the original is still in progress
gets `409`; the same key with a different payload gets `422`. The server must claim the key
atomically, for example with a unique constraint on client plus key, and commit that claim in its
own short transaction before the work starts. If the claim shares the payment's transaction, a
concurrent retry never sees "in progress": in PostgreSQL its insert waits for the first transaction.

## Why choose / why not
- Choose `POST` plus a key when: the server chooses the ids and clients must retry on bad networks,
  such as payments; the cost is a key store, a retention policy and a decision about which outcomes are
  replayed.
- Choose `PUT /payments/{clientChosenId}` instead when: clients can generate ids; it is idempotent by
  definition and needs no key store.
- Answer `303 See Other` to the existing resource when: the payload has a natural business key, such
  as one payment per order.
- Keep keys longer than the client's longest retry window: Stripe may remove a key once it is at
  least 24 hours old, and a reused key then starts a new request.
- Look keys up by client identity plus key: otherwise one client that guesses another's key can read
  the other's stored response.

## Interview angle
- Probed as "design an endpoint that creates a payment, so a mobile client on a bad network can
  retry it safely".
- Common wrong answer: "check whether the key exists, then insert", which is a race, or treating
  the key as a plain response cache.
- Strong answer: start from the failure, then the contract (`201` with `Location`, replay, `409`,
  `422`), claim the key atomically and commit the claim before the charge, scope it per client and
  fingerprint the body; name `PUT` with a client id as the alternative.

## Related
- [[Microservices and messaging MOC]]: the map for "what happens when the other side sees the message
  twice"; this is the request-side answer to that question.
- [[An idempotent consumer records a stable message key in the same transaction as its effect, under a unique constraint]]:
  the broker-side twin; there the marker commits with the effect, while here the claim commits first,
  in its own short transaction, so a concurrent retry can see "in progress", and the stored response
  commits with the local writes.
- [[A Spring ResourceAccessException means no response arrived, so the request's outcome is unknown]]:
  the client side; that note says to retry only an idempotent request or one with a key the server
  deduplicates, and this note is the server contract behind that key.
- [[An idempotent HTTP method repeats the same effect, not the same response, so a retried DELETE may answer 404]]:
  why `POST` needs a key when `PUT` and `DELETE` do not.

Written up in win-interview: backend/docs/rest-api-design.md, sections 2.9 (Idempotency-Key: making a POST safe to retry) and 3.1 (Creating a resource); key retention: backend/docs/idempotency-and-outbox.md, section 2.8 (The HTTP side: an Idempotency-Key store)
