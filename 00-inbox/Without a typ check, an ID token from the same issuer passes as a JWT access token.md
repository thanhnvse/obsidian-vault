---
tags: [security, jwt, oauth, interview]
status: draft
author: claude
up: ["[[Security MOC]]"]
source: "https://www.rfc-editor.org/rfc/rfc9068.html#section-4"
created: 2026-10-01
score: 0.868
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Without a typ check, an ID token from the same issuer passes as a JWT access token

## Core idea
An OpenID Connect ID token and a JWT access token from the same authorization server look
alike: RFC 9068 says the access-token layout is very similar to that of the ID token, and a
resource server has to accept signatures made with any of the keys the server publishes. An ID
token can therefore pass the signature, `iss` and `exp` checks of a resource server. RFC 8725
answers this kind of confusion with explicit typing in the `typ` header, and requires that when
one issuer issues several kinds of JWT, their validation rules are mutually exclusive and reject
JWTs of the wrong kind. RFC 9068 sets `typ` to `at+jwt` for access tokens and requires the
resource server to verify that `typ` is `at+jwt` or `application/at+jwt` and to reject tokens
carrying any other value.

## Why choose / why not
- Enforce `at+jwt` when: the authorization server issues RFC 9068 access tokens and also issues
  ID tokens to browser or mobile clients. In Spring Security 7.1 that means building the
  validator with `JwtValidators.createAtJwtValidator()` (since 6.5), because the default
  `JwtTypeValidator.jwt()` only requires `typ` to be `JWT` or absent.
- Check `aud` as well, not instead: an ID token is addressed to the client, so a strict audience
  check usually rejects it too, but only `typ` separates the two kinds when their audiences
  overlap.
- Don't demand `at+jwt` when: the issuer does not follow RFC 9068 and sends `typ: JWT` or no
  `typ`; the check would reject every token, so keep the rules mutually exclusive with `aud` and
  issuer-specific claims instead, as RFC 8725 asks.

## Interview angle
- Probed as "the single-page app logs in with OpenID Connect and receives an ID token; what stops
  it from calling your API with that token?"
- Common wrong answer: "Nothing needs to; it is signed by our identity provider, so it is valid."
- Strong answer: the signature proves who issued the token, not what kind of token it is. Check
  `typ` = `at+jwt` as RFC 9068 requires, plus `aud`, and know that Spring's default type rule
  does not do this.

## Related
- [[A JWT with a valid signature must still be rejected when exp, iss or aud fail validation]]:
  that note covers the time, issuer and audience checks; this one adds the check on the kind of
  token, which an ID token from the same issuer would otherwise pass.
- [[A JWT validator must pin the accepted algorithms instead of trusting the alg header]]: both
  checks read the JOSE header, but `alg` must never choose how the token is verified, while
  `typ` is compared against the one value this service accepts.
