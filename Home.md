# Home

## Maps
- [[Java backend interview MOC]]
- [[Python MOC]]
- [[JVM MOC]]
- [[Economics MOC]]

New maps proposed by Claude, waiting for review:

```query
[type:moc] [status:draft] path:00-inbox
```

## Approved, waiting to be filed
Set `status: verified` on a note, then ask Claude to "file my approved notes"
(`vault.py promote`): each moves to `10-notes/<area>/`, maps to `20-moc/`.

```query
[status:verified] path:00-inbox
```

## Review queue
Conflicts with a verified note (review these first):

```query
[review:conflict] [status:draft]
```

Ready: every checked criterion passed:

```query
[review:ready] [status:draft]
```

Borderline: Jev was unsure on at least one criterion; worth a look:

```query
[review:borderline] [status:draft]
```

Parked: a criterion failed after revision; reasons are in `score_reasons`:

```query
[review:parked] [status:draft]
```

Other drafts (written by hand, captured with checks skipped, or Jev unavailable):

```query
[status:draft] -[review:conflict] -[review:ready] -[review:borderline] -[review:parked] -[type:moc] -path:templates
```

## Inbox
```query
path:00-inbox
```
