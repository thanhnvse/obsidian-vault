---
tags: [security, tls, https, interview]
status: draft
author: claude
up: ["[[Security MOC]]"]
source: "https://www.rfc-editor.org/rfc/rfc9846.html#section-2.3"
created: 2026-09-30
score: 0.913
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# TLS 1.3 early data can be replayed, so HTTP sends only safe methods in 0-RTT

## Core idea
TLS 1.3 lets a client that resumes with a pre-shared key send application data in its first
flight, before the server has replied. RFC 9846, which obsoletes RFC 8446, says there are no
guarantees of non-replay for this 0-RTT data between connections, because it does not depend on
the server's fresh random value. A copied first flight can therefore be accepted more than once:
server-side anti-replay only guarantees acceptance at most once per server instance. RFC 8470
turns this into HTTP rules: clients may send safe methods in early data and must not send unsafe
ones, and a server that judges the replay risk too high answers 425 (Too Early).

## Why choose / why not
- Choose 0-RTT when: repeat visitors on high-latency links mostly fetch idempotent resources,
  such as pages and static files; the saved round trip is the whole benefit.
- Don't accept early data on state-changing endpoints: a replayed `POST /payments` would run
  twice, so refuse early data for unsafe methods or answer 425 and let the client retry after the
  handshake.
- Keep it off by default: RFC 9846 forbids TLS implementations to enable 0-RTT unless the
  application asks for it, and forbids application protocols to use it without a profile such as
  RFC 8470.

## Interview angle
- Probed as "what is TLS 0-RTT, and when would you turn it off?"
- Common wrong answer: "It is a free speed-up for returning clients."
- Strong answer: early data is sent before the server's random value exists, so it can be
  replayed across connections; only replay-safe requests may use it, and the server answers 425
  when in doubt.

## Related
- [[A TLS 1.3 full handshake authenticates the server and agrees keys in one round trip]]: that
  note explains the one-round-trip handshake and mentions 0-RTT only as a caveat; this note
  explains why early data is replayable and what HTTP does about it.
