---
tags: [security, oauth, browser, interview]
status: draft
author: claude
up: ["[[Security MOC]]"]
source: "https://www.ietf.org/archive/id/draft-ietf-oauth-browser-based-apps-27.html#section-6.1"
created: 2026-09-30
score: 0.84
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A backend for frontend keeps OAuth tokens out of reach of injected JavaScript

## Core idea
In the Backend for Frontend (BFF) pattern of the IETF draft *OAuth 2.0 for Browser-Based
Applications* (revision 27, not yet an RFC), a server-side component is the confidential OAuth
client. It holds the access and refresh tokens and gives the browser only a session cookie,
which the draft requires to be `Secure` and `HttpOnly`. RFC 6265 defines `HttpOnly` as keeping a
cookie away from APIs that expose cookies to scripts. Injected JavaScript can therefore still
send requests through the user's browser, but it finds no token in the page to copy out and use
elsewhere.

## Why choose / why not
- Choose a BFF when: the single-page app handles personal or payment data and a server-side
  component can run on the same site; tokens never reach the browser.
- Budget for CSRF protection when you choose it: the browser sends the session cookie
  automatically, so the draft requires the BFF to implement a CSRF defense.
- Choose in-memory tokens when: no backend component is possible; short lifetimes and refresh
  token rotation limit the damage, but injected script can still read them while the page runs.
- Don't keep tokens in `localStorage`: the draft notes that it is accessible to the entire origin
  and does not protect against malicious JavaScript running there.

## Interview angle
- Probed as "where does your SPA keep its access token?"
- Common wrong answer: "In `localStorage`; the site uses HTTPS, so it is safe."
- Strong answer: HTTPS protects the token on the wire, not in the page. A BFF with a `Secure`,
  `HttpOnly`, `SameSite` session cookie plus CSRF protection keeps the token out of the browser;
  no storage choice stops injected script from acting inside the session.

## Related
- [[A signed JWT is readable by anyone who holds it]]: once a token is stolen, the thief can also
  read every claim in it; this note is about not letting it be stolen from the browser.
- [[Refresh token rotation turns a stolen refresh token into a detectable reuse]]: the fallback
  protection when a public client in the browser must hold a refresh token itself.
