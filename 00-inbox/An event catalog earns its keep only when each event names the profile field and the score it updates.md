---
tags: [customer-data, cdp, event-tracking, tracking-plan]
status: draft
author: claude
up: ["[[Customer data platform MOC]]"]
source: "https://github.com/LEO-CDP/leo-customer360/blob/main/docs/architecture/leo-data-journey-maps.md"
created: 2026-10-08
review: "unjudged"
score_reasons: ["jev unavailable: HTTP 401 after 1 attempt(s): {\"detail\":{\"error_type\":\"authentication_error\",\"message\":\"Cannot authenticate with the server. Please check your API key and try again.\"}}"]
judge_rounds: 1
---
# An event catalog earns its keep only when each event names the profile field and the score it updates

## Core idea
A list of event names with descriptions documents what is tracked but changes nothing downstream.
The catalog becomes useful when every event states which part of the customer profile it updates
and which score it feeds: `purchase` updates transaction history and feeds RFM and lifetime value,
`refund` feeds risk, `message_clicked` feeds the lead score, and a purchase that follows a
recommendation click attributes revenue to the recommender. That mapping tells engineers which
events are load-bearing, because dropping one silently breaks a score; it tells analysts where a
score comes from; and it exposes events that no profile field or score consumes.

## Why choose / why not
- Keep the event-to-profile mapping when: several teams emit events and several others build
  scores from them; it is the contract between the two.
- Don't track an event that no field or score consumes: it costs storage and consent, and adds
  noise to every query.
- Review the mapping when: a score changes meaning, so its source events change with it.

## Interview angle
- Asked as "how would you manage tracking across many product teams?".
- Common wrong answer: a shared spreadsheet of event names.
- Strong answer: a tracking plan where each event lists its properties, its owner and the profile
  fields and scores it feeds, validated at ingestion.

## Related
- [[A tracking event should keep what happened, who did it, where, when and the business context in separate fixed fields]]:
  the envelope that every catalogued event shares.
- [[A persona that is a shared archetype keyed by value tier and lifecycle stage stays countable and explainable]]:
  the scores this mapping feeds are what personas are built from.
- [[Customer data platform MOC]]: the map entry for deciding what to capture.
- Seen in: LEO-CDP/leo-customer360, docs/architecture/leo-data-journey-maps.md, section 6
  "Event Catalog → Customer 360" (read 2026-10-08).
