---
tags: [security, tls, https, interview]
status: draft
author: claude
up: ["[[Security MOC]]"]
source: "https://www.rfc-editor.org/rfc/rfc9846.html#appendix-F.1"
created: 2026-10-01
score: 0.878
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# TLS 1.3 key exchange is ephemeral, so a stolen server private key cannot decrypt recorded sessions

## Core idea
RFC 9846, which obsoletes RFC 8446, records that TLS 1.3 removed the static RSA and static
Diffie-Hellman cipher suites, so all public-key based key exchange mechanisms now provide forward
secrecy. The traffic keys come from ephemeral (EC)DHE key shares, and the server's certificate
private key only signs the handshake in CertificateVerify; it never decrypts key material. RFC
9846 therefore states that if the signature keys are compromised after the handshake is complete,
this does not compromise the session key, as long as the session key and all material that could
recreate it have been erased. Two modes are exceptions: a resumption that uses the pre-shared key
alone, without a new key share, loses forward secrecy for the application data, and the protocol
gives no forward secrecy guarantee for 0-RTT early data.

## Why choose / why not
- Allow only ECDHE suites when TLS 1.2 must stay enabled: with the RSA key transport of TLS 1.2, a
  later leak of the server key decrypts every recorded session that used it.
- Require a new key share on resumption when: recorded traffic must stay confidential even if a
  session-ticket key leaks; PSK-only resumption saves the key-exchange computation but gives up
  forward secrecy.
- Protect and rotate session-ticket keys anyway: RFC 9846 warns that keys which are not erased
  become additional long-term keys that must be protected.

## Interview angle
- Probed as "what is forward secrecy?" or "someone recorded our traffic last year and stole the
  server key today; what can they read?"
- Common wrong answer: "The client encrypts a session key with the server's public key, so the
  private key decrypts it." That describes the TLS 1.2 static RSA key exchange.
- Strong answer: TLS 1.3 traffic keys come from ephemeral key shares and the certificate key only
  signs, so recorded full-handshake sessions stay confidential; then name the exceptions,
  PSK-only resumption and 0-RTT data.

## Related
- [[A TLS 1.3 full handshake authenticates the server and agrees keys in one round trip]]: that
  note shows the key shares in the first flight; this note explains what their being ephemeral
  buys after a key leak.
- [[TLS 1.3 early data can be replayed, so HTTP sends only safe methods in 0-RTT]]: early data is
  the exception in both notes, giving up replay protection there and forward secrecy here.
