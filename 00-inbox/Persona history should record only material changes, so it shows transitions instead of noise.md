---
tags: [customer-data, cdp, persona, data-modelling]
status: draft
author: claude
up: ["[[Customer data platform MOC]]"]
source: "https://github.com/LEO-CDP/leo-customer360/blob/main/customer360-dao/src/leo_customer360_dao/agentic_engines/persona_engine.py"
created: 2026-10-08
review: "unjudged"
score_reasons: ["jev unavailable: HTTP 401 after 1 attempt(s): {\"detail\":{\"error_type\":\"authentication_error\",\"message\":\"Cannot authenticate with the server. Please check your API key and try again.\"}}"]
judge_rounds: 1
---
# Persona history should record only material changes, so it shows transitions instead of noise

## Core idea
Persona scores move a little on every recomputation: one more day since the last visit, one small
purchase. Writing a history row on each run buries the transitions people care about under rows
that differ by a fraction of a point. A history that records a row only when something material
changes, such as the persona name or an overall score that moved by at least a set threshold
(5 points out of 100 by default in one implementation), reads as a list of transitions: became a
champion, slipped to churn risk. Those transitions are what reports and triggers need, and the
current persona row is still versioned on every run.

## Why choose / why not
- Record only material changes when: scores are recomputed often and the history is read by
  people or used to fire triggers.
- Keep full snapshots elsewhere when: you need an audit of every computed value; the transition
  history is not that audit.
- Set the threshold from the score's observed run-to-run noise; below that noise, history fills
  up again.

## Interview angle
- Asked as "how would you store how a customer's segment changes over time?".
- Common wrong answer: insert a row on every recomputation.
- Strong answer: keep the current state versioned and append history only on a material change,
  with the threshold chosen above the measurement noise.

## Related
- [[A persona that is a shared archetype keyed by value tier and lifecycle stage stays countable and explainable]]:
  the assignment whose changes this history records.
- [[Customer data platform MOC]]: the map entry for persona history.
- Seen in: LEO-CDP/leo-customer360, persona_engine.py, `_should_insert_history` (name changed or
  score delta of at least `PERSONA_HISTORY_SCORE_DELTA_THRESHOLD`, default 5.0) (read 2026-10-08).
