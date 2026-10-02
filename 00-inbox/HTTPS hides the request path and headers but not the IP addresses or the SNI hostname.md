---
tags: [security, tls, https, privacy, interview]
status: draft
author: claude
source: "https://www.rfc-editor.org/rfc/rfc9849.html"
created: 2026-09-30
score: 0.813
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# HTTPS hides the request path and headers but not the IP addresses or the SNI hostname

## Core idea
HTTPS protects the content of each request, not the metadata of the connection. TLS makes data
sent after the handshake visible only to the two endpoints, so an on-path observer cannot read the
HTTP method, path, query string, headers or body. The observer still sees the IP addresses in the
packet headers, and the length of the data, which TLS does not hide. The observer also sees the
target hostname, because the Server Name Indication (SNI) extension in the ClientHello is sent in
plaintext unless Encrypted Client Hello (ECH, RFC 9849) is used.

## Why choose / why not
- Keep tokens and personal data out of the query string anyway: the path is encrypted on the wire,
  but full URLs are written to access logs, proxy logs and browser history at the endpoints.
- Enable ECH only when: the hostname itself is sensitive and both the client and the server or CDN
  support it; RFC 9849 warns that plaintext DNS queries and server IP addresses can still reveal
  the destination, so pair it with encrypted DNS.
- Pad TLS records when: response sizes alone would reveal what was fetched; RFC 8446 lets
  endpoints pad records to obscure lengths.

## Interview angle
- Probed as "what can the café Wi-Fi operator see when I open an HTTPS page?"
- Common wrong answer: "Nothing, it's all encrypted", or the reverse, "they can see the full URL."
- Strong answer: split content from metadata. Visible: IP addresses, sizes, timing and the SNI
  hostname. Hidden: method, path, query, headers, cookies and body.

## Related
- [[A TLS 1.3 full handshake authenticates the server and agrees keys in one round trip]]: that
  note shows which handshake messages are encrypted; this note lists what stays visible around
  them.
- [[A signed JWT is readable by anyone who holds it]]: a bearer token in an `Authorization` header
  is hidden from the network by TLS, but not from anyone at an endpoint who can read the header.
