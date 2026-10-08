---
tags: [customer-data, cdp, scoring, money, multi-tenancy]
status: draft
author: claude
up: ["[[Customer data platform MOC]]"]
source: "https://martinfowler.com/eaaCatalog/money.html"
created: 2026-10-08
review: "unjudged"
score_reasons: ["jev unavailable: HTTP 401 after 1 attempt(s): {\"detail\":{\"error_type\":\"authentication_error\",\"message\":\"Cannot authenticate with the server. Please check your API key and try again.\"}}"]
judge_rounds: 1
---
# A score that divides money by a fixed reference value must normalize the amount per currency or per tenant

## Core idea
A financial score such as `min(100, lifetime_value / reference × 100)` silently assumes one
currency and one scale. A reference of 5,000 suits amounts in US dollars; for a tenant that
records amounts in Vietnamese dong, where a single product can cost 25,000,000, almost every
customer reaches the cap, and the score stops telling customers apart while still looking
plausible. It is the problem the Money pattern exists for: amounts in different currencies cannot
be combined or compared without knowing their currency. A score over amounts therefore needs
either conversion to one currency, a reference value per tenant or currency, or a unit-free
measure such as the customer's percentile within the tenant.

## Why choose / why not
- Use percentiles within the tenant when: tenants differ in currency, price level or business
  size; percentiles need no reference value at all.
- Use a fixed reference only when: every amount is in one currency and you know its typical scale.
- Don't store an amount without its currency; the conversion cannot be recovered later.

## Interview angle
- Asked as "how would you score customer value in a multi-tenant, multi-country product?".
- Common wrong answer: "divide lifetime value by a constant and cap at 100".
- Strong answer: carry the currency with every amount and score relative to the tenant's own
  distribution, or convert first and say which rate and date were used.

## Related
- [[A persona that is a shared archetype keyed by value tier and lifecycle stage stays countable and explainable]]:
  the value tier is built from this financial score, so a saturated score collapses the tiers.
- [[Customer data platform MOC]]: the map entry for scoring.
- Seen in: LEO-CDP/leo-customer360, persona_engine.py, `FINANCIAL_CLV_REFERENCE_DEFAULT = 5000`,
  while the sample event in docs/architecture/leo-data-journey-maps.md has a price of 25,000,000;
  the default can be overridden through the persona configuration table (read 2026-10-08).
