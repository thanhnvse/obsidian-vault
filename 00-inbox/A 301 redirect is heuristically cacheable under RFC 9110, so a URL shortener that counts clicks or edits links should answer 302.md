---
tags: [system-design, http, url-shortener, caching, interview]
status: draft
author: claude
up: ["[[System design MOC]]"]
source: "https://www.rfc-editor.org/rfc/rfc9110.html#name-301-moved-permanently"
created: 2026-10-07
review: unjudged
---
# A 301 redirect is heuristically cacheable under RFC 9110, so a URL shortener that counts clicks or edits links should answer 302

## Core idea
RFC 9110 lists 301 among the status codes that are heuristically cacheable (the others are 200, 203,
204, 206, 300, 308, 404, 405, 410, 414 and 501) and says all other codes are not; 302 is not on the
list. A browser or proxy may therefore reuse a 301 without asking the shortener again: repeat visitors
skip the service, click counts fall, and edits and expiry are lost for a client that cached it. A 302
makes the client ask again by default, so every click reaches the service and counts, expiry and
edits work. `307` and `308` are the method-preserving variants and change nothing for a `GET`
redirect. In the write-up's example (100 reads per write, 39 writes per second) a 302 means about
3,900 redirects per second, served from a cache of `code -> long_url` in front of the primary-key
lookup, with click counting on a queue so the redirect never waits.

## Why choose / why not
- Choose 302 when: you count clicks, expire links or let owners edit them, which is the usual product.
- Choose 301 when: the mapping never changes and you do not count clicks; clients may skip your
  server, so the load is lower.
- Don't pick 301 "so it is faster": a client that cached it may keep using the old redirect, so it
  misses a changed or expired link.
- Pay for 302 on the server side: every click is a request, so cache the code and keep click counting
  off the redirect path.

## Interview angle
- Probed as the `302` vs `301` deep dive inside "Design a URL shortener".
- Common wrong answer: "use 301 so it is faster".
- Strong answer: the cache semantics from RFC 9110 and what a cached 301 costs (counts, expiry,
  edits), when 301 is still right, then what 302 costs in load and how a cache absorbs it.

## Related
- [[System design MOC]]: the redirect is the read-heavy hot path of the shortener design, and this
  status-code choice decides how much of it reaches the service.
- [[Cache-aside loads data on a miss and leaves cache consistency to the application]]: the
  server-side cache of code to URL is cache-aside, with a TTL no longer than you can tolerate serving
  a deleted or changed link; a client-cached 301 is a second cache you do not control.
- Written up in win-interview: backend/docs/system-design-walkthrough.md, section 3.5
