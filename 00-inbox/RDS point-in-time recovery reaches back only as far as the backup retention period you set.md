---
tags: [cloud, aws, database, backup, ops, interview]
status: draft
author: claude
up: ["[[Ops and cloud MOC]]"]
source: "https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithAutomatedBackups.html"
created: 2026-10-01
score: 0.837
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# RDS point-in-time recovery reaches back only as far as the backup retention period you set

## Core idea
In the Amazon RDS User Guide (read 2026-10-01), RDS creates automated backups of a DB instance
during its backup window and saves them according to the backup retention period that the customer
specifies. Point-in-time recovery can restore the DB instance to any time within that retention
period, and no further back. So the retention period the customer chooses decides how old a mistake
can be and still be undone from automated backups: a dropped table discovered after the retention
period has passed can no longer be recovered from them.

## Why choose / why not
- Set the retention period from how late a mistake can be found: a data error noticed after the
  window cannot be restored, so keep manual snapshots or a separate backup plan for longer needs.
- Rehearse restores when: the service has a recovery time objective; a point-in-time restore
  creates a new DB instance instead of modifying the source, so only a timed drill shows how long
  the switch to it takes.
- Don't use a point-in-time restore as the undo for a bad release on a live database: every write
  after the restore point is lost unless it is replayed.

## Interview angle
- Probed as "RDS takes backups, so we're covered, right?"
- Common wrong answer: "yes, the managed service handles backups for us."
- Strong answer: backups cover only the retention period we set, a restore creates a new instance,
  and only a tested restore proves that recovery works; both choices are the customer's half.

## Related
- [[The shared responsibility model leaves data and IAM with the customer even on managed services]]:
  the provider runs the backups, but the retention period and the restore test are the customer's
  half of that model.
- [[A Flyway undo migration can reverse a schema change but not lost data, so production databases roll forward]]:
  that note names a backup restore as the last resort after a destructive migration; this one says
  how far back that resort reaches.
