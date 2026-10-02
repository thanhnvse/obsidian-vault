---
tags: [security, jwt, interview]
status: draft
author: claude
source: "https://www.rfc-editor.org/rfc/rfc7519.html"
created: 2026-09-30
score: 0.903
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A signed JWT is readable by anyone who holds it

## Core idea
A signed JWT uses the JWS Compact Serialization: three base64url-encoded parts, the header, the
payload and the signature, joined by periods. Base64url is an encoding that uses no key, so anyone
who holds the token can decode its header and claims. The signature, whether a digital signature
or a MAC, lets the verifier detect that the header or payload was changed; it does not hide them.
RFC 7519 says privacy-sensitive claims need an encrypted JWT (JWE), transport only over TLS, or,
simplest, leaving them out of the token.

## Why choose / why not
- Keep a signed-only JWT when: the claims are identifiers and grants the client may see anyway,
  such as `sub`, `scope` and `exp`.
- Leave personal data out of the token when: the service can look it up by `sub`; a JWT gets
  copied into logs, browser storage and proxies, and every copy is readable.
- Use a nested JWT, signed and then encrypted as a JWE, only when: the data must travel inside the
  token and parties other than the recipient must not read it.
- Use an opaque token with introspection instead when: the client must learn nothing from the
  token at all.

## Interview angle
- Probed as "is a JWT encrypted?" or "can the frontend read the user's roles from the access
  token?".
- Common wrong answer: "It is signed, so nobody can read it."
- Strong answer: separate the properties. The signature gives integrity and origin, TLS protects
  the token in transit, and only JWE hides the claims from whoever holds the token.

## Related
- [[Security MOC]]: the map's Security section asks who can read, forge or replay a
  token; this note answers the "read" part for JWTs.
