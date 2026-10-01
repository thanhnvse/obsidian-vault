---
tags: [moc, security, jwt, interview]
type: moc
status: draft
author: claude
up: ["[[Security MOC]]"]
created: 2026-10-01
---
# JWT MOC

The question behind this map: *what must a service check before it trusts a JWT, and what can it not take back once the token is out?*

## What a signed token is
- [[A signed JWT is readable by anyone who holds it]]: signed is not encrypted

## Validation
- [[A JWT validator must pin the accepted algorithms instead of trusting the alg header]]: the classic algorithm hole
- [[A JWT verifier must take the key from the trusted issuer's JWK Set, never from the token's jku, x5u or jwk header]]: where the verification key must come from
- [[With HS256 every service that can verify a JWT can also forge one]]: why a shared secret does not fit many verifiers
- [[A JWT with a valid signature must still be rejected when exp, iss or aud fail validation]]: a valid signature is not enough
- [[Without a typ check, an ID token from the same issuer passes as a JWT access token]]: confusing an ID token with an access token

## Keys
- [[A JWT signing key is rotated by publishing the new public key in the JWK Set before signing with it]]: the rotation order that keeps valid tokens accepted

## Lifetime and revocation
- [[A self-contained JWT stays valid until it expires unless verifiers check revocation state]]: the revocation trade-off
- [[Refresh token rotation turns a stolen refresh token into a detectable reuse]]: how short-lived access tokens stay usable without a long-lived risk

## In the browser
- [[A backend for frontend keeps OAuth tokens out of reach of injected JavaScript]]: where a browser app should keep its tokens
