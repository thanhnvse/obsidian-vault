---
tags: [database, sql, schema-design, normalization, interview]
status: draft
author: claude
up: ["[[Database MOC]]"]
source: "https://www.db-book.com/slides-dir/PDF-dir/ch7.pdf"
created: 2026-10-01
score: 0.879
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Splitting a table is lossless when the columns the two parts share are a key of one part

## Core idea
Normalising splits one table into smaller tables that are joined back on the columns they share.
The split is lossless when that join returns exactly the original rows, nothing missing and nothing
extra; otherwise it is lossy. A split of R into R1 and R2 is lossless when the shared columns,
R1 ∩ R2, functionally determine all of R1 or all of R2, that is, when they are a key of one part.
That condition is sufficient, and it is also necessary when all the table's constraints are
functional dependencies. Splitting a flat order-line table into
`orders(order_id, order_date, customer_id)` and `order_line(order_id, product_code, quantity)`
shares `order_id`, the key of `orders`, so each line joins back to exactly one order. Splitting it
into `(order_id, customer_city)` and `(customer_city, product_code, quantity)` shares
`customer_city`, a key of neither part, so the join pairs each order with the lines of every other
order from the same city: rows that never existed. On PostgreSQL 15.19 the third-normal-form split
of a five-row table joined back to exactly the five rows, with `EXCEPT` empty in both directions,
while the split on the city produced rows that were not in the original.

## Why choose / why not
- Split on a determinant when: normalising; move the columns it determines into a new table keyed
  by it, and leave it behind as a foreign key, which makes the split lossless by construction.
- Check with `EXCEPT` in both directions when: migrating existing data into the new tables; two
  empty differences show the join gives back exactly the original rows.
- Don't split on a shared descriptive column, such as a city, status or name, when: it is a key of
  neither part; the join invents rows and no constraint reports it.

## Interview angle
- Probed as "how do you know your decomposition is correct?".
- Common wrong answer: "each new table is in third normal form and no column was dropped, so it is
  correct."
- Strong answer: name the lossless-join condition, that the shared columns must be a key of one
  part, show a split that breaks it, and prove the real one by joining back.

## Related
- [[Third normal form removes update anomalies by making every non-key column depend only on the key]]:
  it says which columns to move out of a table, and this note says how to split them off without
  inventing rows.
- [[Cardinality decides where a relationship's foreign key goes]]: the determinant left behind in
  the split becomes the foreign key on the many side, and that note says why it goes there.
