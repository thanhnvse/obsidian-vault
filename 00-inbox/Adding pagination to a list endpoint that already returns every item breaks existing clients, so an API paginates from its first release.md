---
tags: [rest, api-design, pagination, http, interview]
status: draft
author: claude
up: ["[[Microservices and messaging MOC]]"]
source: "https://google.aip.dev/158"
created: 2026-10-07
review: unjudged
---
# Adding pagination to a list endpoint that already returns every item breaks existing clients, so an API paginates from its first release

## Core idea
Google's API Improvement Proposal on pagination calls adding pagination to an existing list a
breaking change: a client that used to get all 75 items now gets the first 50 "and does not know to
advance pagination". Azure's guidelines say the same about adding paging later. Nothing fails
loudly: the old client sees one page and takes it for the whole list. So the first release
already carries the contract, `GET /orders?pageSize=50&pageToken=...`, with `nextPageToken` in the
response and absent on the last page. `pageSize` is optional, with a documented default and
maximum.

## Why choose / why not
- Paginate a list endpoint from the first release: the paging parameters are then in the
  contract before any client exists.
- The cost: a cursor cannot jump to page N and needs a unique sort key and a matching index.
  Offset pages still have a place for small admin lists where a user jumps to page 7.
- Keep the token opaque: base64 of the last id is not opaque, so clients build their own tokens
  and the format can never change.
- "We'll add pagination when the table gets big" is the red flag: the symptom is support tickets
  about missing data after a release.

## Interview angle
- Probed as "How would you paginate a large collection that changes while clients page through
  it?", and inside "How do you version an API? What is a breaking change?", where adding
  pagination is on the list.
- Common wrong answer: "We'll add pagination when the table gets big."
- Strong answer: pagination exists from day one, as an opaque keyset cursor with a `pageSize`
  default and maximum, the token absent on the last page, the same filters and authorisation on
  every page, and a documented guarantee.

## Related
- [[Microservices and messaging MOC]]: the other REST contract notes sit here, and this is the
  evolution side of the same contract: the write-up counts pagination, versioning and retries
  as the contracts that are hard to add later.
- [[A retried POST is safe only when the server claims its Idempotency-Key atomically before the work and replays the stored result]]:
  the retry contract, the third of those three.
- [[Keyset pagination stays fast on deep pages because it does not read the skipped rows]]: the
  query behind the cursor; this note is about what the API exposes and why it must exist at
  release.
- [[OFFSET pagination skips or repeats rows when rows before the page change between requests]]:
  why the cursor is not an offset.

Written up in win-interview: backend/docs/rest-api-design.md, section 2.7
