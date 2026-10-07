---
tags: [observability, metrics, micrometer, interview]
status: draft
author: claude
up: ["[[Ops and cloud MOC]]"]
source: "https://prometheus.io/docs/practices/naming/"
created: 2026-10-07
review: unjudged
---
# Each distinct combination of metric tag values is a separate time series, so unbounded values such as a user id do not belong in tags

## Core idea
Every distinct combination of tag values is another meter in Micrometer's registry and another
time series in the backend, and a meter stays in the registry until something removes it. In a lab,
a tag that took a user id produced 1 000 meters for 1 000 users, while a tag from a bounded set of
three outcomes stayed at three meters however many requests arrived. Spring Boot's
`http.server.requests` timer follows the rule: its `uri` tag is the route template, so `/greet/1`
and `/greet/2` are one series, `uri=/greet/{id}`. A request id or a user id belongs on a trace or in a log line, which
are made for high cardinality (the number of distinct values). A `MeterFilter` that caps a tag, such
as `maximumAllowableTags("requests", "user", 10, MeterFilter.deny())`, keeps the first 10 values and
drops the rest, so it is a safety net: the data it drops is the data you wanted to see.

## Why choose / why not
- Tag with a bounded set when: the values are a route template, an outcome or a status class; the
  meter count stays constant whatever the traffic.
- Put the id on a trace or in a log line when: you need to search by user or by request; those
  signals are built for it.
- Add a `MeterFilter` cap when: a tag could still take unbounded (không giới hạn) values by mistake;
  it is a safety net, and the call site still gets a no-op meter and does not fail.
- Don't enable histograms on every timer: each bucket is one more series per tag combination, as in
  [[Averaging per-instance p95 values rarely makes sense, so a fleet-wide p95 is computed from summed histogram buckets]].

## Interview angle
- Probed as "what is cardinality, and how did you keep it under control?"
- Common wrong answer: "add the user id as a tag so we can search by user."
- Strong answer: each tag combination is a series held in memory and in the backend, so tag only
  with bounded values, put ids on traces and logs, keep a cap as a safety net, and check the scrape
  for `uri="/orders/123"` where `/orders/{id}` should be.

## Related
- [[Averaging per-instance p95 values rarely makes sense, so a fleet-wide p95 is computed from summed histogram buckets]]:
  the histogram that note recommends multiplies the series per tag combination, so the two rules
  limit each other.
- [[Ops and cloud MOC]]: a metrics backend that slows down under thousands of series is an
  operations failure, so tag discipline belongs on the map of how a service is run.
- Written up in win-interview: backend/java/docs/observability.md, section 2.3
