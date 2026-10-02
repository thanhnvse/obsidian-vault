---
tags: [security, jwt, oauth, interview]
status: draft
author: claude
source: "https://www.rfc-editor.org/rfc/rfc7009.html"
created: 2026-09-30
score: 0.907
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A self-contained JWT stays valid until it expires unless verifiers check revocation state

## Core idea
A resource server that validates a self-contained JWT locally checks the signature and the `exp`
claim without contacting the authorization server, so it cannot learn that the token was revoked.
RFC 7519 defines `exp` as the time on or after which the JWT must not be accepted, and RFC 6819
notes that token revocation is more difficult with self-contained tokens than with handles.
RFC 7009 describes two designs: backend interaction between authorization server and resource
server when immediate revocation is needed, or short-lived access tokens refreshed with a refresh
token, which limits how long a revoked grant keeps working. Immediate revocation therefore needs a
lookup of shared state on each request, such as token introspection (RFC 7662), which reports
whether a token is still active, or a denylist of revoked `jti` values.

## Why choose / why not
- Choose short-lived access tokens plus a revocable refresh token when: a logout or a disabled
  account may take effect within one access-token lifetime; verification stays local and
  stateless.
- Choose introspection or a `jti` denylist when: access must stop at once, such as for a token
  reported stolen; accept a lookup (usually cached) on every request, so the verifier is no
  longer stateless.
- Don't issue long-lived self-contained access tokens: a leaked one keeps working until `exp`, and
  RFC 6819 lists a short expiration time as a protection against token leak and replay.

## Interview angle
- Probed as "how do you log a user out when you use JWTs?" or "an account is disabled; how soon
  does its token stop working?"
- Common wrong answer: "Delete the token from local storage." That removes the client's copy, not
  the token's validity.
- Strong answer: name the trade-off, stateless verification versus immediate revocation, then
  choose a bound: access tokens that live minutes, refresh tokens revoked at the authorization
  server, and a denylist only for the cases that cannot wait.

## Related
- [[A JWT validator must pin the accepted algorithms instead of trusting the alg header]]: that
  note covers what a validator checks in the token itself; this note covers what no local check
  can see, a revocation that happened after issue.
- [[Security MOC]]: this note answers the "who can replay this?" question of the
  Security section for bearer tokens.
