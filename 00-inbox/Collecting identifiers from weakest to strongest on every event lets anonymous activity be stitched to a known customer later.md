---
tags: [customer-data, cdp, event-tracking, identity-resolution]
status: draft
author: claude
up: ["[[Customer data platform MOC]]"]
source: "https://www.rudderstack.com/docs/event-spec/standard-events/identify/"
created: 2026-10-08
review: "unjudged"
score_reasons: ["jev unavailable: HTTP 401 after 1 attempt(s): {\"detail\":{\"error_type\":\"authentication_error\",\"message\":\"Cannot authenticate with the server. Please check your API key and try again.\"}}"]
judge_rounds: 1
---
# Collecting identifiers from weakest to strongest on every event lets anonymous activity be stitched to a known customer later

## Core idea
A visitor is anonymous long before they are known: first a session id, then a first-party
anonymous id kept in a cookie or local storage, then a device id or fingerprint, and only after a
login or a form a user id, email or phone. If every event carries every identifier the client has
at that moment, rather than only the strongest one, the trail stays connected: the event where an
anonymous id and a user id appear together, typically the login, lets identity resolution attach
all the earlier anonymous events to the known customer. If the client drops the anonymous id after
login, or sends only the user id, the pre-login history is orphaned. Weak identifiers link
anonymous activity to a person; they are not proof that two known customers are the same person.

## Why choose / why not
- Send the whole ladder on every event when: you analyse pre-login funnels or attribute campaigns
  to people who register later.
- Don't merge two known customers on a weak identifier alone (a shared tablet, a fingerprint); use
  it to link anonymous events, and require a strong identifier to merge profiles.
- Keep the anonymous id after login when: you need to join the session that converted; reset it
  only at logout on a shared device.

## Interview angle
- Asked as "how do you know that the anonymous visitor last week is the customer who just bought?".
- Common wrong answer: "use the user id", which only exists after login.
- Strong answer: carry the anonymous id on every event and link it to the user id at the moment
  both are present, then resolve history backwards.

## Related
- [[A tracking event should keep what happened, who did it, where, when and the business context in separate fixed fields]]:
  this note fills in the "who" field of that envelope.
- [[A golden customer record must keep each source record's link with its match method and score, so a wrong merge can be explained and undone]]:
  a weak-identifier link is exactly the kind of match whose method and score must be recorded.
- [[Customer data platform MOC]]: the map entry for identity in captured events.
- Seen in: LEO-CDP/leo-customer360, customer360-event-api/core/schemas.py, which accepts
  `session_id`, `anonymous_id`, `device_id`, `device_fingerprint` and `user_id` on each batch and
  each event (read 2026-10-08).
