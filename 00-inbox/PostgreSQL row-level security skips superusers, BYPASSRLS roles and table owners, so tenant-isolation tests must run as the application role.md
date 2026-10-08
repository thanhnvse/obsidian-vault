---
tags: [database, postgresql, security, row-level-security, multi-tenancy, interview]
status: draft
author: claude
up: ["[[Database MOC]]"]
source: "https://www.postgresql.org/docs/current/ddl-rowsecurity.html"
created: 2026-10-08
review: "unjudged"
score_reasons: ["jev unavailable: HTTP 401 after 1 attempt(s): {\"detail\":{\"error_type\":\"authentication_error\",\"message\":\"Cannot authenticate with the server. Please check your API key and try again.\"}}"]
judge_rounds: 1
---
# PostgreSQL row-level security skips superusers, BYPASSRLS roles and table owners, so tenant-isolation tests must run as the application role

## Core idea
Row-level security policies apply only to roles that are subject to them. Superusers and roles
with the `BYPASSRLS` attribute always bypass row security, and a table's owner bypasses it too
unless the table is altered with `FORCE ROW LEVEL SECURITY`. An application or a test suite that
connects as `postgres`, the user in many sample `.env` files, therefore sees every tenant's rows,
and tenant-isolation tests pass while proving nothing. The application should connect as a role
that is neither a superuser nor the table owner, and the policy should fail closed: when the
tenant setting is missing, it should match no rows.

## Why choose / why not
- Use row-level security when: tenant isolation must hold even if one application query forgets
  its `WHERE tenant_id = ...`; it is defense in depth, not a replacement for scoping in code.
- Run migrations as the owner and the application as a separate role when: you enable row-level
  security; otherwise add `FORCE ROW LEVEL SECURITY` to every protected table.
- Don't trust: a green isolation test until you know which role it connected as.

## Interview angle
- Asked as "how would you enforce tenant isolation in a shared PostgreSQL schema?".
- Common wrong answer: "enable row-level security and add a policy", while the application still
  connects as the table owner.
- Strong answer: a policy, an application role that is neither owner nor superuser, a fail-closed
  tenant setting, and isolation tests that run as that role.

## Related
- [[A response cache in front of row-level security leaks across tenants unless the tenant is part of every cache key]]:
  row-level security also never sees requests that a cache answers.
- [[Database MOC]]: the map entry for what PostgreSQL enforces on its own.
- Seen in: LEO-CDP/leo-customer360, customer360-database/database-schema.sql warns that superusers
  and BYPASSRLS roles bypass the policies, while .env.example connects as `postgres` (read 2026-10-08).
