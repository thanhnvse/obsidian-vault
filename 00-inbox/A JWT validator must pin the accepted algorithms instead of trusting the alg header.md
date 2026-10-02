---
tags: [security, jwt, spring, interview]
status: draft
author: claude
source: "https://www.rfc-editor.org/rfc/rfc8725.html"
created: 2026-09-30
score: 0.703
review: "parked"
score_reasons: ["why_choose: 0.39 (fail)", "numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A JWT validator must pin the accepted algorithms instead of trusting the alg header

## Core idea
The `alg` header of a JWT is written by whoever produced the token, so a validator that takes its
verification algorithm from that header lets the sender decide how the token is checked. RFC 8725
records libraries that accepted a token whose `alg` was changed to `none` without checking any
signature, and libraries that checked a token relabelled from RS256 to HS256 by using the RSA
public key as the HMAC secret. RFC 8725 therefore requires libraries to let the caller specify the
supported algorithms and to use no others. It also requires the library to ensure that `alg`
names the algorithm actually used, and that each key is used with exactly one algorithm.

## Why choose / why not
- Always pin: configure the accepted algorithms next to the key in the decoder. In Spring Security
  7.1, `NimbusJwtDecoder` trusts only RS256 unless `jws-algorithms` or the decoder builder says
  otherwise.
- When migrating algorithms, for example from RS256 to ES256: publish a new key with its own
  `kid` for the new algorithm instead of letting one key serve both, because RFC 8725 allows one
  algorithm per key.
- Don't hand-roll JWT verification: a switch on the `alg` header rebuilds the exact flaw RFC 8725
  describes; use a maintained library and configure its allow-list instead.

## Interview angle
- Probed as "what is the `alg: none` problem, and how does your service prevent it?"
- Common wrong answer: "The library reads `alg` from the header and verifies with it, so any
  algorithm the library supports is fine."
- Strong answer: the header is sender-controlled input; the configured allow-list and the key
  decide the algorithm, and the header only has to match them.

## Related
- [[A signed JWT is readable by anyone who holds it]]: that note shows the header is plain
  base64url that anyone can read and rewrite, which is why the validator must not take the
  algorithm from it.
- [[Security MOC]]: this note answers the "who can forge this?" question of the
  Security section for JWTs.
