---
tags: [security, oauth, jwt, interview]
status: draft
author: claude
up: ["[[Security MOC]]"]
source: "https://www.rfc-editor.org/rfc/rfc9700.html#section-4.14.2"
created: 2026-09-30
score: 0.899
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Refresh token rotation turns a stolen refresh token into a detectable reuse

## Core idea
RFC 9700, the OAuth 2.0 security best current practice, requires authorization servers to detect
refresh token replay for public clients, either with sender-constrained refresh tokens or with
refresh token rotation. With rotation, every refresh returns a new refresh token and invalidates
the previous one, while the authorization server remembers that both belong to the same grant.
If an attacker and the legitimate client both hold a copy, whichever of them uses it second
presents an invalidated token, and that tells the authorization server the token leaked. The
server cannot tell which party is the attacker, so it revokes the active refresh token, and the
legitimate user has to sign in again.

## Why choose / why not
- Choose rotation when: the client is public, such as a single-page app or a mobile app that
  cannot keep a client secret, and sender-constrained tokens (mutual TLS or DPoP) are not
  available; RFC 9700 accepts either of the two.
- Don't rely on rotation alone when: the client may send two refreshes with the same token at
  once, for example from two tabs or a retry; the second request looks exactly like theft and
  ends the session, so serialise refreshes in the client.
- Add an inactivity expiry anyway: RFC 9700 says refresh tokens should expire when the client has
  not used them for some time, which bounds how long a token nobody rotates stays usable.

## Interview angle
- Probed as "a refresh token is stolen from a mobile app; how does the authorization server find
  out?"
- Common wrong answer: "It can't; a refresh token is valid until it expires."
- Strong answer: rotation with reuse detection. Name the cost, a forced re-login for the real
  user, and the alternative, binding the token to the client with mutual TLS or DPoP.

## Related
- [[A self-contained JWT stays valid until it expires unless verifiers check revocation state]]:
  that note relies on short access tokens plus a revocable refresh token; this note covers how
  the refresh token itself is protected once it is the long-lived credential.
