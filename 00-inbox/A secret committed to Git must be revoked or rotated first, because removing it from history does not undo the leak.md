---
tags: [security, secrets, git, interview]
status: draft
author: claude
up: ["[[Security MOC]]"]
source: "https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository"
created: 2026-10-01
score: 0.907
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A secret committed to Git must be revoked or rotated first, because removing it from history does not undo the leak

## Core idea
Once a commit with a secret is pushed, every clone and fork made since then holds a copy, and a
later commit that deletes the line leaves the value in history. GitHub's documentation therefore
says that when the sensitive data is a secret, the first step is to revoke and/or rotate it; once
it is revoked or rotated it can no longer be used for access, and rewriting history to remove it
may not even be warranted. Rewriting history has costs of its own: it changes the hashes of the
commit that introduced the data and of every later commit, a collaborator with a clone from
before the rewrite can push the data back, and the commit stays accessible in any fork that
contains it.

## Why choose / why not
- Revoke or rotate at once when: a credential reached any remote, even for minutes; assume it was
  copied, because clones, forks and caches cannot be recalled.
- Rewrite history as well when: the data cannot be revoked, such as personal data, or a policy
  requires its removal; coordinate it, because every collaborator must replace their old clone.
- Don't treat "we removed it in the next commit" as a fix: the value is still in history for
  everyone who can clone the repository.

## Interview angle
- Probed as "a developer pushed the production database password to the repository; what do you
  do first?"
- Common wrong answer: "Delete it in a new commit", or "force-push a rewritten history and we are
  fine."
- Strong answer: revoke or rotate the credential first, so the leaked value is useless; check
  access logs for use of the old value; then decide whether rewriting history is worth its cost.

## Related
- [[HashiCorp Vault dynamic secrets are generated per client with a lease and can be revoked]]: a
  dynamic credential can be revoked through its lease alone; this note is about the static secret
  that must be rotated by hand once it leaks.
- [[A Kubernetes Secret is only base64-encoded and is stored unencrypted in etcd by default]]: that
  note warns against committing a Secret manifest, whose base64 values are plaintext to anyone who
  clones; this note says what to do once that has happened.
