---
tags: [database, sql, schema-design, constraints, postgresql, interview]
status: draft
author: claude
up: ["[[Database MOC]]"]
source: "https://www.postgresql.org/docs/15/ddl-constraints.html"
created: 2026-10-01
score: 0.893
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A PostgreSQL UNIQUE constraint accepts several NULLs unless it is declared NULLS NOT DISTINCT

## Core idea
A `UNIQUE` constraint in PostgreSQL treats two NULL values as not equal by default. A nullable
column with `UNIQUE`, such as `customer.tax_id`, therefore accepts any number of rows whose
`tax_id` is NULL, and enforces uniqueness only among the rows that have a value. PostgreSQL 15
added `UNIQUE NULLS NOT DISTINCT`, which treats NULLs as equal, so the second row with a NULL
violates the constraint; before PostgreSQL 15, NULL entries were always treated as distinct. On
PostgreSQL 15.19 a default `UNIQUE` column accepted several NULLs, and a `NULLS NOT DISTINCT`
column rejected the second NULL with SQLSTATE `23505`.

## Why choose / why not
- Keep the default when: NULL means "not known yet" and many rows may lack the value, such as an
  optional tax id; uniqueness then applies only to the rows that have one.
- Declare `NULLS NOT DISTINCT` when: the rule is "at most one row without a value", such as one
  open contract per customer with `UNIQUE NULLS NOT DISTINCT (customer_id, ended_at)`, and the
  server runs PostgreSQL 15 or later; on older servers a partial unique index,
  `CREATE UNIQUE INDEX ON contract (customer_id) WHERE ended_at IS NULL`, enforces the same rule.
- Make the column `NOT NULL` instead when: the value is the business key that identifies the row,
  such as a login email; a key that may be missing identifies nothing.

## Interview angle
- Probed as "email is UNIQUE, so can two users share one?", then "and if email is nullable?".
- Common wrong answer: "UNIQUE means at most one row per value, NULL included."
- Strong answer: NULLs are distinct by default, so a nullable unique column holds many NULLs;
  either make the key `NOT NULL`, or name `NULLS NOT DISTINCT` together with its version,
  PostgreSQL 15.

## Related
- [[Cardinality decides where a relationship's foreign key goes]]: a one-to-one relationship is a
  foreign key made `UNIQUE`, and when that foreign key is nullable, this NULL rule decides how many
  rows may have no partner.
- [[A partial index cannot serve a generic plan whose bind parameter decides the predicate]]: it
  explains partial indexes, and a partial unique index is how servers older than PostgreSQL 15
  allow at most one NULL.
