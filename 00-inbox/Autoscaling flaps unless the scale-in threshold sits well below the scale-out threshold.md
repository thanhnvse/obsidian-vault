---
tags: [system-design, interview, autoscaling, scaling, cloud]
status: draft
author: claude
up: ["[[System design MOC]]"]
source: "https://learn.microsoft.com/en-us/azure/architecture/best-practices/auto-scaling"
created: 2026-10-01
score: 0.883
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Autoscaling flaps unless the scale-in threshold sits well below the scale-out threshold

## Core idea
Flapping is an autoscaler adding and removing instances back and forth. It happens when removing an instance pushes the load on the remaining ones over the scale-out threshold. Microsoft's autoscaling guidance gives the example of two instances with a scale-out threshold of 80% CPU and a scale-in threshold of 60%: at 85% a third instance is added; when the load later falls to 60% across three instances, removing one would put the remaining two at 90%, so the scale-in would trigger an immediate scale-out. Azure Monitor autoscale calculates that distribution before scaling in and skips the scale-in, so the fleet may never shrink as expected. The guidance's remedy is an adequate margin between the scale-out and scale-in thresholds. Autoscale rules also act on a value aggregated over time, by default an average, rather than on instantaneous values, which keeps the system from reacting too quickly or oscillating.

## Why choose / why not
- Size the gap from the fleet size when: the fleet is small; removing one of N instances raises per-instance load by N/(N-1), so with 3 instances and an 80% scale-out threshold, scale in only below about 53% (example numbers).
- Add a scale-in stabilization period when: the load is noisy; it waits for the lower load to persist before removing capacity.
- Don't narrow the gap to save cost when: new instances take minutes to start; every flap pays a start-up during which the new instance serves nothing.

## Interview angle
- Probed as "the fleet keeps scaling in and out every few minutes; why?", or "set the scaling rules for this service".
- Common wrong answer: "scale out at 70%, scale in at 65%", thresholds chosen without checking the load after one instance is removed.
- Strong answer: compute the per-instance load after removing one instance, keep the scale-in threshold below it, and add minimum and maximum bounds plus a stabilization period.

## Related
- [[Scale queue workers on the age of the oldest message, not on queue depth]]: that note chooses the metric to scale on; this one is about the thresholds set on whatever metric is chosen.
- [[Stateless services scale out, while scaling up one machine stops at a hardware limit]]: an autoscaler can add and remove instances freely only because they are stateless; flapping is the cost of doing that too eagerly.
