---
tags: [security, jwt, interview]
status: draft
author: claude
up: ["[[Security MOC]]"]
source: "https://www.rfc-editor.org/rfc/rfc7515.html#section-10.5"
created: 2026-09-30
score: 0.913
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# With HS256 every service that can verify a JWT can also forge one

## Core idea
HS256 protects a JWT with an HMAC, a message authentication code that uses the same secret key
to compute the tag and to check it. RFC 7515 notes that a MAC key has to be in the hands of every
entity that computes or checks it, so a valid MAC only proves that one of the key holders produced
the token. If an authorization server and three APIs share one HS256 secret, each API can mint
tokens that the other two accept, and one leaked copy compromises all of them. With RS256 or
ES256 only the issuer holds the private key; verifiers receive public keys, which can check a
signature but cannot create one.

## Why choose / why not
- Choose HS256 only when: the service that issues the token is also the only one that verifies
  it, with a random key of at least 256 bits; RFC 8725 forbids using a human-memorable password
  directly as an HMAC key.
- Choose RS256 or ES256 when: a second service verifies tokens; the issuer publishes its public
  keys in a JWK Set and verifiers pick one by `kid`, without ever being able to sign.
- Don't share an HS256 secret across teams or with third parties: every copy is a signing key,
  and rotating it means changing every copy at once.

## Interview angle
- Probed as "HS256 or RS256 for tokens that several microservices verify?"
- Common wrong answer: "HS256 is faster and just as secure, as long as the secret is long."
- Strong answer: the question is who holds which key; with a MAC every verifier is also a
  potential issuer, so asymmetric signing is the default once more than one service verifies.

## Related
- [[A JWT validator must pin the accepted algorithms instead of trusting the alg header]]: that
  note covers the RS256-to-HS256 confusion, where a validator treats a public key as an HMAC
  secret; this note explains why an HMAC key must stay secret and is shared by every verifier.
- [[A signed JWT is readable by anyone who holds it]]: that note is about who can read a token;
  this one is about who can create a token that verifiers accept.
