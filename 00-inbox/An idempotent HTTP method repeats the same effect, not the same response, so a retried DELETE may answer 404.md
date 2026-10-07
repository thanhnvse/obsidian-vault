---
tags: [rest, http, api-design, idempotency, interview]
status: draft
author: claude
up: ["[[Microservices and messaging MOC]]"]
source: "https://www.rfc-editor.org/rfc/rfc9110#section-9.2.2"
created: 2026-10-07
review: unjudged
---
# An idempotent HTTP method repeats the same effect, not the same response, so a retried DELETE may answer 404

## Core idea
RFC 9110 calls a method idempotent when the intended effect on the server of several identical
requests is the same as for a single one. It is about the effect, not the response: a first
`DELETE` answers `204` and a repeat answers `404`, yet both leave the resource deleted, so `DELETE`
is still idempotent. `GET`, `HEAD`, `OPTIONS`, `PUT` and `DELETE` are idempotent; `POST` and `PATCH` are
not. The definition covers what the client asked for, so a server may still log every request or
send an email on every `PUT`; a retried `PUT` then sends two, and that is the server's bug. The
property matters for retries: an idempotent request can be repeated automatically after a failure
before the client read the response, while a client should not automatically retry a non-idempotent
method without a way to know it is safe, and a proxy must not.

## Why choose / why not
- Answer `204` to a repeated `DELETE` when: a retry should look like the success it is; Azure's
  guidelines do this even when the URL names a resource that does not exist.
- Answer `404` when: a caller with the wrong id should find out; then document that a `404` on a
  retried `DELETE` is fine.
- Let the client name the resource and use `PUT` when: the call must be repeatable without extra
  machinery, since a repeated `PUT` ends in the same state.
- Don't retry a `POST` after a timeout when: it carries no idempotency key; a retried
  `POST /orders` creates two orders. See [[A retried POST is safe only when the server claims its Idempotency-Key atomically before the work and replays the stored result]].

## Interview angle
- Probed as "is `DELETE` idempotent if the second call returns `404`?" or "what is the difference
  between safe and idempotent?".
- Common wrong answer: "idempotent means the same response every time."
- Strong answer: idempotent is about the effect on the server, and the response may differ; then
  the consequence for retries, and your own choice of `204` or `404` for a repeated `DELETE`, documented.

## Related
- [[Microservices and messaging MOC]]: the map for "what happens when the other side sees the
  message twice"; this is the HTTP-method form of that question.
- [[A Spring ResourceAccessException means no response arrived, so the request's outcome is unknown]]:
  that note says a timeout leaves the outcome unknown; this note says which requests may then be
  repeated, by method.
- [[A retried POST is safe only when the server claims its Idempotency-Key atomically before the work and replays the stored result]]:
  what to add to `POST`, the method this note leaves non-idempotent.

Written up in win-interview: backend/docs/rest-api-design.md, sections 2.2 (Method semantics: safe, idempotent, cacheable), 2.4 (Status codes: the first digit is the contract; the `204` or `404` choice) and 2.10 (Consequences you can derive from the mechanism; the retried `POST /orders`)
