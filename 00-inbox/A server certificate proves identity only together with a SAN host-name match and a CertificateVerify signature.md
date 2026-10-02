---
tags: [security, tls, https, interview]
status: draft
author: claude
up: ["[[Security MOC]]"]
source: "https://www.rfc-editor.org/rfc/rfc9110.html#section-4.3.4"
created: 2026-10-01
score: 0.84
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A server certificate proves identity only together with a SAN host-name match and a CertificateVerify signature

## Core idea
A certificate is public data: anyone can obtain a server's certificate chain and present it, so a
chain that validates up to a trusted root only binds a public key to a name. Two further checks
turn that binding into proof of the server's identity. The requested host name must match a
DNS-ID in the leaf certificate's subjectAltName extension: RFC 9110 forbids clients to use a CN-ID
reference identity, and RFC 9525 says the server identity can only be expressed in the
subjectAltNames extension. And the server's CertificateVerify message, a signature over the
handshake transcript made with the private key of the leaf certificate, gives explicit proof that
the server possesses that key. Without the name check, a valid certificate for another host would
be accepted; without CertificateVerify, a copied certificate would be enough.

## Why choose / why not
- Keep all three checks on in every client, test environments included: RFC 9110 says ignoring
  the server's identity leaves a connection open to active attack, so a trust-all `TrustManager`
  or a disabled host-name verifier encrypts traffic to whoever answers.
- Add the private CA to the client's trust material when: services use certificates from an
  internal CA; in Spring Boot that is an SSL bundle under `spring.ssl.bundle.*`, not a disabled
  check.
- List every served host name in the SAN when: one certificate serves several hosts; a name that
  appears only in the subject common name does not count for current clients.

## Interview angle
- Probed as "how does the client know it is talking to the real server and not to a copy?"
- Common wrong answer: "The certificate is valid, so the server is who it says it is."
- Strong answer: three checks, not one. The chain ends at a trust anchor, the host name is in the
  SAN, and CertificateVerify proves possession of the private key; the certificate alone is
  public.

## Related
- [[A TLS 1.3 full handshake authenticates the server and agrees keys in one round trip]]: that
  note places CertificateVerify in the handshake flow; this note separates it from the chain and
  name checks and says what each one stops.
- [[HTTPS hides the request path and headers but not the IP addresses or the SNI hostname]]: the
  host name the client sends in SNI there is normally the same name it then expects to find in
  the certificate's SAN here.
