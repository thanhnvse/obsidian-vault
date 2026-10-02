---
tags: [database, sql, schema-design, interview]
status: draft
author: claude
source: "https://www.postgresql.org/docs/18/ddl-constraints.html"
created: 2026-09-30
score: 0.839
review: "borderline"
score_reasons: ["atomic: 0.48 (borderline)", "numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 2
---
# Cardinality decides where a relationship's foreign key goes

## Core idea
A foreign key value references exactly one row of the other table, so the foreign key of a
relationship belongs on the side whose rows each have exactly one partner. In a one-to-many
relationship that is the "many" side: each order row carries its `customer_id`. In a
many-to-many relationship neither side qualifies, so the relationship becomes its own table, a
join table with one foreign key to each side and usually the pair as its primary key, as in the
PostgreSQL 18 documentation's `order_items (product_no, order_id)` example. Attributes of the
relationship itself, such as the quantity of a product on an order, then live in that join
table.

## Why choose / why not
- Put the foreign key on the many side when: each child row belongs to exactly one parent, such
  as an order line to its order; declare it with `REFERENCES` so the database rejects orphans.
- Use a join table when: both sides can have many partners, such as products and orders, or the
  link carries its own data; the composite primary key stops the same pair being linked twice.
- Don't store the keys as a comma-separated string or an array column when: you need to join on
  them or keep them valid; a foreign key constraint cannot check the elements of a list.

## Interview angle
- Probed as "design the tables for orders and products"; the interviewer checks that the foreign
  key lands on the many side and that `order_items` appears without prompting.
- Common wrong answer: "a unidirectional JPA `@OneToMany` is a foreign key on the child table";
  Hibernate maps a unidirectional `@OneToMany` through a link table, and in a bidirectional
  association the `@ManyToOne` child side owns the foreign key.
- Strong answer: state the rule (the key goes where there is exactly one partner), then add that
  PostgreSQL does not index the referencing column for you, so deleting a parent scans the
  child table unless you add that index.

## Related
- [[Database MOC]]: schema design is the first Database question, asked before
  indexing or isolation, so the map lists this note as the entry point for that section.
