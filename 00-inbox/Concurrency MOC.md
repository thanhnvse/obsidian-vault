---
tags: [moc, concurrency, database, interview]
type: moc
status: draft
author: claude
up: ["[[Java backend interview MOC]]"]
created: 2026-09-30
---
# Concurrency MOC

The question behind this map: *what happens when two requests touch the same data at once?*

## Lost updates and locking
- [[A synchronized block cannot prevent a lost update between two application instances]]: why the race moves to the database
- [[A version column detects a lost update at write time instead of blocking the other writer]]: optimistic locking
- [[SELECT FOR UPDATE makes competing writers queue on the row until the lock holder commits]]: pessimistic locking
