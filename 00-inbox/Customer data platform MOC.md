---
tags: [moc, customer-data, cdp]
type: moc
status: draft
author: claude
created: 2026-10-08
---
# Customer data platform MOC

The question behind this map: *how does scattered customer data become one profile, a score and an
audience?*

## Capture
- [[A tracking event should keep what happened, who did it, where, when and the business context in separate fixed fields]]: the event contract every consumer relies on
- [[Collecting identifiers from weakest to strongest on every event lets anonymous activity be stitched to a known customer later]]: what each event must carry so anonymous activity can be joined later
- [[An event catalog earns its keep only when each event names the profile field and the score it updates]]: how an event earns its place in the catalog

## Assemble
- [[A golden customer record must keep each source record's link with its match method and score, so a wrong merge can be explained and undone]]: what makes a merged profile explainable and reversible

## Score and personas
- [[A persona that is a shared archetype keyed by value tier and lifecycle stage stays countable and explainable]]: personas as shared, countable archetypes
- [[Persona history should record only material changes, so it shows transitions instead of noise]]: keeping persona history readable
- [[A score that divides money by a fixed reference value must normalize the amount per currency or per tenant]]: scoring money across currencies and tenants

## AI on customer data
- [[An LLM that names a customer segment should receive aggregated, non-PII statistics only]]: what a model may see when it names a segment
