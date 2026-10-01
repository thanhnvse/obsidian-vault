---
tags: [moc, security, tls, interview]
type: moc
status: draft
author: claude
up: ["[[Security MOC]]"]
created: 2026-10-01
---
# TLS MOC

The question behind this map: *what does TLS prove and hide, and where does the encryption end?*

## Handshake and identity
- [[A TLS 1.3 full handshake authenticates the server and agrees keys in one round trip]]: the HTTPS flow
- [[A server certificate proves identity only together with a SAN host-name match and a CertificateVerify signature]]: what a certificate alone does not prove
- [[TLS 1.3 key exchange is ephemeral, so a stolen server private key cannot decrypt recorded sessions]]: forward secrecy
- [[TLS 1.3 early data can be replayed, so HTTP sends only safe methods in 0-RTT]]: the price of 0-RTT

## What an observer sees
- [[HTTPS hides the request path and headers but not the IP addresses or the SNI hostname]]: what HTTPS hides and what it leaves visible

## Where TLS ends
- [[Edge TLS termination leaves the hop to the Pods in plaintext unless the proxy re-encrypts or passes TLS through]]: where encryption stops inside a cluster
- [[Behind a TLS-terminating proxy, forwarded headers are client input unless the proxy wrote them]]: what the application can trust after termination
