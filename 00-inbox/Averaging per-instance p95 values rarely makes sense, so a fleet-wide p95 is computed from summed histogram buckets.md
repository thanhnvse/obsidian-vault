---
tags: [observability, metrics, micrometer, latency, interview]
status: draft
author: claude
up: ["[[Ops and cloud MOC]]"]
source: "https://prometheus.io/docs/practices/histograms/"
created: 2026-10-07
review: unjudged
---
# Averaging per-instance p95 values rarely makes sense, so a fleet-wide p95 is computed from summed histogram buckets

## Core idea
A p95 is computed from one instance's own requests, so per-instance p95 values do not combine by
arithmetic. In a Micrometer 1.12.1 lab, one instance had 900 requests at 100 ms and another 100
requests at 1 000 ms: their p95 values average to about 550 ms, yet the true p95 of all 1 000
requests is 1 000 ms, so the average understates (đánh giá thấp) the tail. With 990 requests at 10 ms and 10 at
2 000 ms the average is about 1 005 ms and the true p95 is 10 ms, so it can overstate it too. Bucket
counts do add: publish cumulative histogram buckets, sum them across instances and compute the
quantile from the sum, which is what Prometheus' `histogram_quantile()` does. The same logic makes
a weighted mean correct (190 ms for the first case) and a mean of means wrong (550 ms).

## Why choose / why not
- Publish percentile histograms when: a latency SLO is judged across several instances or tags;
  bucket counts add, so any backend can compute the quantile at query time.
- Keep client-side percentiles
  (`management.metrics.distribution.percentiles.http.server.requests=0.95`) when: you read one
  instance and accept an approximate value; the `quantile="0.95"` series cannot be aggregated.
- Don't enable histograms on every timer: each bucket is one more series per tag combination, so
  the series count and the scrape time grow; use them on the timers you alert on, such as
  `http.server.requests`.
- Don't read a histogram quantile finer than its boundaries: 100 requests at 150 ms landed in the
  1 000 ms bucket in the lab, so put boundaries around the SLO.

## Interview angle
- Probed as "why can't you average the p95 across instances, and how do you get a fleet-wide p95?"
- Common wrong answer: "average the p95 of each pod", or alert on that average; the lab shows it
  wrong in both directions.
- Strong answer: counts and sums add but quantiles do not; sum the histogram buckets across
  instances, compute the quantile from the sum, and say that precision depends on the bucket
  boundaries.

## Related
- [[A canary's errors are diluted in whole-service metrics, so it must be compared with a control group]]:
  another case where a whole-service aggregate blurs a minority; there a 5% canary failing 20% of its
  requests shows as a 1% error rate, here an averaged p95 understates a slow instance.
- [[Ops and cloud MOC]]: latency alerts decide whether an operator learns that a service is
  breaking, so the aggregation rule behind them belongs on the map of how a service is run.
- Written up in win-interview: backend/java/docs/observability.md, section 2.3
