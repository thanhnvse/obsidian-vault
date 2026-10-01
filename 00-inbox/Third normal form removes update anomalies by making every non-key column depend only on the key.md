---
tags: [database, sql, schema-design, normalization, interview]
status: draft
author: claude
source: "https://learn.microsoft.com/en-us/previous-versions/troubleshoot/microsoft-365/microsoft-365-apps/access/database-normalization-description"
created: 2026-09-30
score: 0.858
review: "borderline"
score_reasons: ["atomic: 0.54 (borderline)", "numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 2
---
# Third normal form removes update anomalies by making every non-key column depend only on the key

## Core idea
A table is in third normal form (3NF) when every non-key column depends on the table's key and
on no other non-key column. The classic violation stores a fact about something else on the
row, such as an advisor's room number on every student row: the room depends on the advisor,
not on the student. The same fact is then stored once per student, so changing it means
updating every copy, and missing one leaves rows that contradict each other; that is the
update anomaly. Moving the dependent column into its own table, keyed by the column it depends
on, stores the fact once, so a change touches one row and no copy can disagree.

## Why choose / why not
- Normalise to 3NF when: the schema backs transactional writes; each fact has exactly one row
  to update, so a partial or concurrent update cannot leave contradictory copies behind.
- Keep a repeated value when: the copy must not follow its source, such as the unit price on an
  invoice line, which records the price at the time of sale; no anomaly exists because the
  copy depends on the invoice line, not on the catalogue.
- Leave a dependency in place when: the dependent value practically never changes, such as a
  city next to its postal code; with no update to protect, the extra table only adds a join.

## Interview angle
- Probed as "what is 3NF and why does it matter?"; the interviewer wants the update anomaly it
  prevents, not a recited definition.
- Common wrong answer: "3NF means no duplicated values"; the invoice-line price repeats the
  catalogue price, yet it depends on the line and belongs where it is.
- Strong answer: "every non-key column depends on the key and nothing but the key", then one
  update anomaly and the table split that removes it.

## Related
- [[Cardinality decides where a relationship's foreign key goes]]: splitting a transitive
  dependency into its own table creates a new one-to-many relationship, and that note says
  which side gets the foreign key.
