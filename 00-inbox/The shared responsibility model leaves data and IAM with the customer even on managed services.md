---
tags: [cloud, aws, security, ops, interview]
status: draft
author: claude
source: "https://aws.amazon.com/compliance/shared-responsibility-model/"
created: 2026-09-30
score: 0.887
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# The shared responsibility model leaves data and IAM with the customer even on managed services

## Core idea
In AWS's shared responsibility model, AWS is responsible for "security of the cloud": the
hardware, software, networking and facilities that run AWS Cloud services. The customer is
responsible for "security in the cloud", and how much work that is depends on the services the
customer selects. On Amazon EC2, an Infrastructure as a Service offering, the customer manages
the guest operating system including its security patches, the software installed on the
instance, and the security group firewall. On abstracted services such as Amazon S3 and
DynamoDB, AWS operates the infrastructure, operating system and platform, but the customer still
manages its data including encryption options, classifies its assets, and sets IAM permissions.

## Why choose / why not
- Choose a managed service when: the team does not want to own operating system patching and
  platform operation; that layer moves to the provider.
- Choose a self-managed instance when: you need control at the operating system level, such as
  kernel settings or custom agents, and accept owning its patches and hardening.
- Don't treat "managed" as "secured": data and access permissions stay yours on every service,
  so a bucket left open to the public is still the customer's mistake.

## Interview angle
- Probed as "we moved to a managed database, so security is the provider's job now?"; answer
  with the line between the provider's layer and yours for that service type.
- Common wrong answer: "a managed service makes the provider responsible for security."
- Strong answer: name what always stays with the customer whatever the service, its data with
  its encryption choices and its IAM permissions, then say what the chosen service adds on top.

## Related
- [[Ops and cloud MOC]]: this answers the cloud half of the Ops and cloud cluster, the
  question of what a platform takes off the team's hands.
