---
tags: [security, jwt, interview]
status: draft
author: claude
up: ["[[Security MOC]]"]
source: "https://www.rfc-editor.org/rfc/rfc8725.html#section-3.10"
created: 2026-10-01
score: 0.847
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A JWT verifier must take the key from the trusted issuer's JWK Set, never from the token's jku, x5u or jwk header

## Core idea
The `jku` and `x5u` headers of a JWS are URLs that refer to the signing key or its certificate,
and the `jwk` header carries the public key itself. Like every header they are written by whoever
produced the token, so a key reached through them proves only that someone holds the matching
private key, not who that someone is. RFC 7515 says the key management technique used to obtain
public keys must authenticate the origin of the key; otherwise it is unknown what party signed
the message. RFC 8725 adds that blindly following a `jku` or `x5u` URL could result in
server-side request forgery, and that applications should validate or sanitise the `kid` used
for key lookup so that it does not create SQL or LDAP injection. A verifier therefore uses `kid` only
as a hint to choose among the keys it already trusts from the configured issuer's JWK Set.

## Why choose / why not
- Configure the key source in the verifier, never per token: in Spring Security set `issuer-uri`
  or `jwk-set-uri`, so the decoder takes its keys from the issuer's published JWK Set.
- Treat `kid` as untrusted input when: keys are looked up in a database, a directory or a file
  system; use a parameterised lookup, never string concatenation into SQL, LDAP or a path.
- Don't accept an embedded `jwk` even in tests: a check that passes with any key the token brings
  along verifies nothing, and such shortcuts tend to reach production.

## Interview angle
- Probed as "the token header says which key signed it; why not just use that key?"
- Common wrong answer: "Fetch the key from the `jku` URL; that is what the header is for."
- Strong answer: until the signature verifies with a key you chose, every byte of the token is
  untrusted. The verifier fixes the key source to the trusted issuer's JWK Set, uses `kid` only
  to pick among those keys, and validates `kid` like any other input.

## Related
- [[A JWT validator must pin the accepted algorithms instead of trusting the alg header]]: the
  same rule for the other half of verification; that note keeps the header from choosing the
  algorithm, this one keeps it from choosing the key.
- [[With HS256 every service that can verify a JWT can also forge one]]: that note shows that
  verifiers receive the issuer's public keys through a JWK Set; this note explains why that set,
  and not the token, must supply the key.
