---
tags: [cloud, aws, database, high-availability, ops, interview]
status: draft
author: claude
up: ["[[Ops and cloud MOC]]"]
source: "https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZSingleStandby.html"
created: 2026-10-01
score: 0.904
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# An RDS Multi-AZ standby gives failover, not read capacity

## Core idea
In the Amazon RDS User Guide (read 2026-10-01), a Multi-AZ DB instance deployment automatically
provisions and maintains a synchronous standby replica in a different Availability Zone. The
primary DB instance is synchronously replicated across Availability Zones to the standby to provide
data redundancy. If a planned or unplanned outage of the Multi-AZ DB instance results from an
infrastructure defect, RDS automatically switches to the standby replica in the other Availability
Zone. The guide states that this high availability option is not a scaling solution for read-only
scenarios: the standby replica cannot serve read traffic, and read-only traffic needs a Multi-AZ
DB cluster or a read replica instead.

## Why choose / why not
- Choose a Multi-AZ instance when: the database must survive the loss of an instance or an
  Availability Zone with an automatic failover, and the standby must hold every committed write.
- Add a read replica when: read load is the problem; a replica serves queries, but it is
  asynchronous and can return stale data.
- Budget for: a standby instance that you run and pay for but that adds no query capacity.

## Interview angle
- Probed as "we have Multi-AZ, so can the reporting queries go to the standby?"
- Common wrong answer: "Multi-AZ doubles our read capacity."
- Strong answer: the standby is synchronous and exists for failover only; read scaling needs read
  replicas or a Multi-AZ DB cluster, whose readers serve reads.

## Related
- [[Read replicas scale reads but serve stale data while replication lags]]: the read-scaling
  option this note contrasts with; a replica takes reads but lags, while the Multi-AZ standby is
  synchronous and takes no reads.
- [[The shared responsibility model leaves data and IAM with the customer even on managed services]]:
  the provider runs the failover, but turning Multi-AZ on is the customer's configuration.
