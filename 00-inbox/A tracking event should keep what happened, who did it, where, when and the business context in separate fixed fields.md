---
tags: [customer-data, cdp, event-tracking, schema-design, system-design]
status: draft
author: claude
up: ["[[Customer data platform MOC]]"]
source: "https://github.com/cloudevents/spec/blob/main/cloudevents/spec.md"
created: 2026-10-08
review: "unjudged"
score_reasons: ["jev unavailable: HTTP 401 after 1 attempt(s): {\"detail\":{\"error_type\":\"authentication_error\",\"message\":\"Cannot authenticate with the server. Please check your API key and try again.\"}}"]
judge_rounds: 1
---
# A tracking event should keep what happened, who did it, where, when and the business context in separate fixed fields

## Core idea
Give every tracking event one envelope with a fixed field per question: an event name for what
happened, a profile or visitor identifier for who did it, a source block (channel, platform,
integration) for where, a timestamp with its time zone for when, and one free-form object for the
business context such as product, price or cart. Every consumer (identity resolution, analytics,
segmentation) then reads the same top-level fields without knowing which source sent the event,
and anything source-specific lives inside the context object, so adding a source does not change
the contract. CloudEvents standardises the same split with its `type`, `source`, `subject`, `time`
and `data` attributes. A `schema_version` field lets the contract evolve without breaking old
consumers.

## Why choose / why not
- Use one fixed envelope when: many sources feed many consumers; each consumer otherwise needs
  per-source parsing.
- Don't add business attributes as new top-level fields per source: every consumer has to learn
  them; put them in the context object.
- Don't encode context in the event name (`add_to_cart_summer_banner`); the campaign belongs in
  its own block, so `add_to_cart` stays one countable event.

## Interview angle
- Asked as "design the event schema for a tracking pipeline".
- Common wrong answer: one flat JSON per source, designed screen by screen.
- Strong answer: a versioned envelope (id, name, who, source, time) plus a context object, with
  the id doubling as the idempotency key for retries.

## Related
- [[Collecting identifiers from weakest to strongest on every event lets anonymous activity be stitched to a known customer later]]:
  the "who" field is not one identifier but a ladder of them.
- [[An event catalog earns its keep only when each event names the profile field and the score it updates]]:
  the envelope says what happened; the catalog says what it changes.
- [[Customer data platform MOC]]: the map entry for capturing customer events.
- Seen in: LEO-CDP/leo-customer360, docs/architecture/leo-data-journey-maps.md, section 5
  "Canonical LEO Event Schema" (read 2026-10-08).
