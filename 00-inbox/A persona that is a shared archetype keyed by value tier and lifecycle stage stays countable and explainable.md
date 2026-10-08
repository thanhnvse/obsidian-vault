---
tags: [customer-data, cdp, persona, segmentation, scoring]
status: draft
author: claude
up: ["[[Customer data platform MOC]]"]
source: "https://github.com/LEO-CDP/leo-customer360/blob/main/customer360-dao/src/leo_customer360_dao/agentic_engines/persona_engine.py"
created: 2026-10-08
review: "unjudged"
score_reasons: ["jev unavailable: HTTP 401 after 1 attempt(s): {\"detail\":{\"error_type\":\"authentication_error\",\"message\":\"Cannot authenticate with the server. Please check your API key and try again.\"}}"]
judge_rounds: 1
---
# A persona that is a shared archetype keyed by value tier and lifecycle stage stays countable and explainable

## Core idea
A persona can be a description written for each customer, or an archetype that many customers
share. The shared form computes a few banded attributes and builds the persona key from them, for
example a value tier (the mean of the financial and loyalty scores: champion from 80, high value
from 60, growth potential from 35, otherwise standard) combined with the lifecycle stage
(prospect, lead, customer, VIP, dormant, churn risk). Many customers then point to one archetype
row, which holds the persona's name and summary. That makes personas countable ("how many dormant
champions?"), usable directly as audiences, explainable (the tier and stage that put a customer
there), and cheap to name, because a name is written once per archetype rather than once per
customer.

## Why choose / why not
- Use shared archetypes when: personas drive segmentation, reporting or campaign targeting.
- Use a per-customer narrative when: one person at a time reads one profile, such as a sales rep
  preparing a call.
- Don't key archetypes on raw scores; band them, and leave a gap between the score that enters a
  band and the score that leaves it, or customers near a boundary flap between personas on every
  recomputation.

## Interview angle
- Asked as "how would you build customer personas in a CDP?".
- Common wrong answer: "an LLM writes a persona for every customer".
- Strong answer: score a few dimensions, band them into tiers, key a shared archetype on the bands
  and the lifecycle stage, and keep the scores next to the persona so it can be explained.

## Related
- [[Persona history should record only material changes, so it shows transitions instead of noise]]:
  how an archetype assignment changes over time without flooding history.
- [[An LLM that names a customer segment should receive aggregated, non-PII statistics only]]:
  what the model may see when it names an archetype.
- [[A score that divides money by a fixed reference value must normalize the amount per currency or per tenant]]:
  the financial score that feeds the value tier.
- [[Autoscaling flaps unless the scale-in threshold sits well below the scale-out threshold]]:
  the same flapping around a single boundary, fixed the same way, with a gap between thresholds.
- [[Customer data platform MOC]]: the map entry for personas.
- Seen in: LEO-CDP/leo-customer360, persona_engine.py, `compute_customer_value_tier` and
  `compute_persona_code` (domain, value tier and lifecycle stage), with the archetype held in
  `cdp_persona_archetypes` (read 2026-10-08).
