---
tags: [security, jwt, spring, interview]
status: draft
author: claude
source: "https://www.rfc-editor.org/rfc/rfc8725.html"
created: 2026-09-30
score: 0.893
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A JWT with a valid signature must still be rejected when exp, iss or aud fail validation

## Core idea
A valid signature proves that a JWT came from the holder of the signing key and was not changed;
it does not prove that this service is meant to accept the token now. That decision rests on the
claims. RFC 7519 requires rejecting a JWT on or after its `exp` time, and rejecting it when an
`aud` claim is present that does not identify the processing service. RFC 8725 requires checking
that the signing keys belong to the issuer named in `iss`, and requires every relying party to
validate `aud` whenever one issuer issues JWTs for more than one relying party.

## Why choose / why not
- Configure issuer and audience explicitly: in Spring Security 7.1 the resource server validates
  `exp`, `nbf` and `iss` by default, but `aud` only when the Boot `audiences` property or a custom
  validator is set.
- Skip the audience check only when: the issuer issues tokens for this one service alone; RFC 8725
  makes it mandatory as soon as one issuer serves several relying parties, because otherwise a
  token meant for one of them is accepted by the others.

## Interview angle
- Probed as "the gateway and the billing API trust the same identity provider; what stops a token
  issued for the gateway from calling billing directly?"
- Common wrong answer: "The signature is valid, so the token is trusted."
- Strong answer: the signature proves origin; `exp`, `iss` and `aud` decide whether this service,
  at this moment, is the intended recipient. In Spring Boot that means `issuer-uri` plus
  `audiences`.

## Related
- [[A JWT validator must pin the accepted algorithms instead of trusting the alg header]]: that
  note covers the check that comes before the signature; this note covers the checks after it, and
  together they give the full validation order.
- [[A self-contained JWT stays valid until it expires unless verifiers check revocation state]]:
  `exp` is the only end of life a local check can enforce, and that note covers what to do when it
  is not soon enough.
