---
tags: [security, tls, https, interview]
status: draft
author: claude
source: "https://www.rfc-editor.org/rfc/rfc8446.html"
created: 2026-09-30
score: 0.817
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A TLS 1.3 full handshake authenticates the server and agrees keys in one round trip

## Core idea
In a TLS 1.3 full handshake, key agreement and server authentication share a single round trip.
The client's ClientHello already carries Diffie-Hellman key shares, and the server's reply starts
with a ServerHello carrying its own ephemeral share, from which both sides derive handshake keys.
The rest of the server's flight, EncryptedExtensions, Certificate, CertificateVerify and Finished,
is encrypted under those keys. CertificateVerify is a signature over the handshake transcript made
with the certificate's private key, which proves the server holds that key; detailed certificate
chain validation is left to RFC 5280. The client then sends its own Finished and can send
application data, one round trip after its ClientHello.

## Why choose / why not
- Choose TLS 1.3 wherever both ends support it: a new connection costs one round trip of
  handshake before the first request can be sent.
- Don't turn on 0-RTT early data to save that round trip for non-idempotent requests: RFC 8446
  says 0-RTT data has no guarantee of non-replay between connections, so a replayed
  `POST /payments` can run twice.
- Keep TLS 1.2 enabled only when: a named legacy client still needs it.

## Interview angle
- Probed as "what happens between typing an https:// URL and the first byte of the response?"
- Common wrong answer: "The client encrypts a session key with the server's public key." That is
  the TLS 1.2 static RSA key exchange, which TLS 1.3 removed.
- Strong answer: key shares in the first flight, the server authenticated by a signature over the
  transcript, and the client's first request right after its Finished, one round trip in.

## Related
- [[A signed JWT is readable by anyone who holds it]]: a bearer JWT relies on the channel this
  handshake sets up to stay confidential in transit, because the token itself hides nothing.
- [[Security MOC]]: this note opens the HTTPS thread of the Security section.
