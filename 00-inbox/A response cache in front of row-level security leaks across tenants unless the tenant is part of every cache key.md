---
tags: [security, multi-tenancy, caching, row-level-security, interview]
status: draft
author: claude
up: ["[[Security MOC]]"]
source: "https://www.postgresql.org/docs/current/ddl-rowsecurity.html"
created: 2026-10-08
review: "unjudged"
score_reasons: ["jev unavailable: HTTP 401 after 1 attempt(s): {\"detail\":{\"error_type\":\"authentication_error\",\"message\":\"Cannot authenticate with the server. Please check your API key and try again.\"}}"]
judge_rounds: 1
---
# A response cache in front of row-level security leaks across tenants unless the tenant is part of every cache key

## Core idea
Row-level security filters rows inside the database, query by query, using the tenant set for that
session or transaction. A response cache sits before the database: on a cache hit no query runs,
so no policy runs either. If the cache key is built only from the route and its parameters, tenant
B requesting the same URL as tenant A receives A's cached response. The tenant, and any other
authorization input that changes the response such as the caller's role, must be part of every
cache key, taken from the authenticated request context rather than from whichever arguments a
handler happens to declare.

## Why choose / why not
- Add the tenant to the key in one central place (middleware, or a context variable the cache
  layer reads) when: many handlers share one caching decorator; a per-handler rule is easy to miss.
- Don't cache when: the response depends on per-user permissions that cannot be folded into the key.
- Test with the cache enabled and two tenants requesting the same URL; a test that stubs the cache
  out cannot catch this leak.

## Interview angle
- Asked as "you added Redis caching to a multi-tenant API; what can leak?".
- Common wrong answer: "nothing, the database enforces tenant isolation with row-level security".
- Strong answer: a cache hit never reaches the database, so the cache has to enforce the same
  isolation through its key.

## Related
- [[PostgreSQL row-level security skips superusers, BYPASSRLS roles and table owners, so tenant-isolation tests must run as the application role]]:
  the other way row-level security silently stops protecting tenants.
- [[Cache-aside loads data on a miss and leaves cache consistency to the application]]: the
  application, not the database, decides what goes into the cache and under which key.
- [[Security MOC]]: the map entry for tenant isolation.
- Seen in: LEO-CDP/leo-customer360, customer360-api/core/cache.py, which adds the tenant to the key
  only when the handler receives a `Request` object (read 2026-10-08).
