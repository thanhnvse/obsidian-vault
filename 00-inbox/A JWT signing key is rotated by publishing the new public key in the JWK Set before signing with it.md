---
tags: [security, jwt, interview]
status: draft
author: claude
up: ["[[Security MOC]]"]
source: "https://docs.spring.io/spring-security/reference/servlet/oauth2/resource-server/jwt.html"
created: 2026-10-01
score: 0.882
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A JWT signing key is rotated by publishing the new public key in the JWK Set before signing with it

## Core idea
A verifier picks the verification key by the token's `kid` among the keys it fetched from the
issuer's JWK Set, and it caches that set: Spring Security 7.1.1 caches it in memory for 5 minutes
by default and picks up new keys as the authorization server makes them available. OpenID
Connect Core 1.0 describes rotation as adding new keys to the JWK Set, starting to sign with a new
key and signalling the change through `kid`, and keeping recently decommissioned keys in the set
for a reasonable period. The safe order is therefore: publish the new public key next to the old
one, start signing with the new key only once it is published, and remove the old key only after
the last token it signed has expired. A token signed with a key that is not yet published fails
verification, and removing the old key too early rejects tokens that are still within their
lifetime.

## Why choose / why not
- Wait one JWK cache lifetime between publishing and signing when: some verifiers refresh their
  key cache only on a timer instead of re-fetching on an unfamiliar `kid`; in Spring Security the
  default is 5 minutes.
- Don't keep the old key published when it leaked: remove it at once and accept that every token
  it signed is rejected, because a compromised key must not verify anything.
- Expect a harder rotation with HS256: there is no public key to publish, so every verifier must
  receive the new shared secret before the issuer uses it.

## Interview angle
- Probed as "how do you rotate the token-signing key without logging everybody out?"
- Common wrong answer: "Replace the key at the identity provider and restart the services."
- Strong answer: publish, wait, sign, retire. The new public key goes into the JWK Set first,
  signing switches after verifier caches can have it, and the old key stays until the
  longest-lived token it signed has expired; name your framework's JWK cache time.

## Related
- [[With HS256 every service that can verify a JWT can also forge one]]: that note explains why
  verifiers get public keys from a JWK Set; this note covers how that set changes without an
  outage.
- [[A self-contained JWT stays valid until it expires unless verifiers check revocation state]]:
  removing a signing key is the one revocation a local check does see, since every token the key
  signed then fails verification, which is why it is kept for a leaked key.
