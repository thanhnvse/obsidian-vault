---
tags: [customer-data, cdp, llm, privacy, security]
status: draft
author: claude
up: ["[[Customer data platform MOC]]"]
source: "https://genai.owasp.org/llmrisk/llm022025-sensitive-information-disclosure/"
created: 2026-10-08
review: "unjudged"
score_reasons: ["jev unavailable: HTTP 401 after 1 attempt(s): {\"detail\":{\"error_type\":\"authentication_error\",\"message\":\"Cannot authenticate with the server. Please check your API key and try again.\"}}"]
judge_rounds: 1
---
# An LLM that names a customer segment should receive aggregated, non-PII statistics only

## Core idea
An LLM can turn a segment's numbers into a readable name and summary, such as "loyal high-value
shoppers at risk of churning". For that it needs aggregates: score bands, value tier, lifecycle
stage, counts and averages. It does not need names, emails, phone numbers or addresses. Sending only
aggregated, non-personal inputs keeps personal data out of a third party's logs and retention,
and removes a prompt-injection path through free-text profile fields that customers can write
themselves. A deterministic fallback name keeps the pipeline working when the model is slow or
down. Hashed identifiers do not count as non-personal: an unsalted hash of a phone number can be
reversed by hashing every possible number.

## Why choose / why not
- Send aggregates when: the model's job is to describe a group, not to act on one person.
- Don't send raw or hashed identifiers "just for context"; they add no meaning to a group name and
  add the data to every log on the way.
- Keep a deterministic fallback when: the name is part of a batch job that must not fail or stall
  on the model.

## Interview angle
- Asked as "what data would you send to an LLM to summarise customer segments?".
- Common wrong answer: "the profiles of the segment, anonymised by hashing the email".
- Strong answer: aggregated, non-personal statistics only, a fallback when the call fails, and no
  free-text fields that customers control.

## Related
- [[A persona that is a shared archetype keyed by value tier and lifecycle stage stays countable and explainable]]:
  naming one archetype instead of each customer is what keeps the model's input aggregate.
- [[Customer data platform MOC]]: the map entry for AI on customer data.
- Seen in: LEO-CDP/leo-customer360, persona_engine.py, whose design notes state that persona
  naming and summaries are given only non-PII, already-computed statistics (read 2026-10-08).
