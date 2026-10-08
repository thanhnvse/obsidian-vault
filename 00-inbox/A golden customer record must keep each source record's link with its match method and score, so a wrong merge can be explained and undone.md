---
tags: [customer-data, cdp, identity-resolution, data-modelling, auditability]
status: draft
author: claude
up: ["[[Customer data platform MOC]]"]
source: "https://github.com/LEO-CDP/leo-customer360/blob/main/README.md"
created: 2026-10-08
review: "unjudged"
score_reasons: ["jev unavailable: HTTP 401 after 1 attempt(s): {\"detail\":{\"error_type\":\"authentication_error\",\"message\":\"Cannot authenticate with the server. Please check your API key and try again.\"}}"]
judge_rounds: 1
---
# A golden customer record must keep each source record's link with its match method and score, so a wrong merge can be explained and undone

## Core idea
Identity resolution merges records from many sources (web, app, POS, CRM) into one master profile,
the golden record. Every merge is a judgement of some strength: an exact email match is strong, a
shared device cookie is weak. If the raw source records stay untouched, and a separate link table
records for each one which master it joined, by which rule (match method) and with what score, plus
a merge history, the golden record becomes explainable ("why are these two people one profile?")
and reversible: a wrong merge is undone by relinking the raw records, because nothing was
overwritten. Without that lineage, a bad merge silently mixes two people's personal data and can
only be repaired by hand.

## Why choose / why not
- Keep raw records immutable and links in their own table when: merges are probabilistic or
  sources disagree; a wrong merge is then a data change, not a data loss.
- Record which source won each master field (survivorship) when: sources conflict on the same
  attribute, such as two addresses.
- Don't auto-merge on one weak identifier; send weak-only matches to review, or link them as
  related profiles instead of merging.

## Interview angle
- Asked as "design customer deduplication across channels" or "how would you undo a bad merge?".
- Common wrong answer: update the surviving row in place and delete the duplicate.
- Strong answer: raw records plus a scored, method-tagged link table and a merge history, so a
  merge can be explained, audited and reversed.

## Related
- [[Collecting identifiers from weakest to strongest on every event lets anonymous activity be stitched to a known customer later]]:
  the identifiers on each event are the evidence these links are built from.
- [[Customer data platform MOC]]: the map entry for assembling one customer profile.
- Seen in: LEO-CDP/leo-customer360, customer360-dao/src/leo_customer360_dao/models/identity.py,
  with `cdp_raw_profiles_stage`, `cdp_profile_links` (`match_method`, `match_score`) and
  `cdp_profile_merge_history` (read 2026-10-08).
