---
tags: [system-design, url-shortener, hashing, interview]
status: draft
author: claude
up: ["[[System design MOC]]"]
source: "https://en.wikipedia.org/wiki/Birthday_problem"
created: 2026-10-07
review: unjudged
---
# A truncated hash used as a short code collides far sooner than its code space suggests, by the birthday bound

## Core idea
A short code made from a truncated hash, or from random characters, collides when two URLs get the
same code. With N possible codes, the chance of any collision among n codes reaches probability p at
about `n = sqrt(2 N ln(1/(1-p)))`, the standard birthday approximation. For 7 base62 characters,
N = 62^7, about 3.5 trillion, that is about 266,000 links for a 1 % chance and about 2.2 million for
50 %. For 4 characters N = 62^4 = 14.8 million and the first collision is expected near 4,800 URLs;
in the write-up's fixed list it came at the 957th. So a hash or random code with no uniqueness check
is wrong at any real size: the primary key on `code` is the arbiter (the second insert fails with
`23505`), and the creation path needs a written retry, such as another salt for a hash.

## Why choose / why not
- Choose a counter in base62 when: you want the shortest codes and public links may be enumerable
  (dò từng cái một được); it never collides, but codes are guessable and a sequence leaves gaps
  after a rollback.
- Choose a truncated hash when: you want idempotent creation, since the same URL gives the same
  first code; resolve a collision with a salt and a retry.
- Choose a random code when: links must not be enumerable; one insert collides with probability rows
  divided by space, about 0.17 % at 6 billion rows in 62^7 codes (example), and each collision costs a
  retry.
- Don't rely on "collisions are unlikely": keep the primary key and the retry path.
- At a very high write rate a single counter is a hot spot; each instance can reserve a block of ids.

## Interview angle
- Probed as "how do you generate the short code, and what about collisions?"
- Common wrong answer: "hash the URL and take the first 7 characters; collisions are unlikely."
- Strong answer: compare counter, hash and random; give the birthday number, name the primary key as
  the arbiter with a retry path, and mention range allocation for scale.

## Related
- [[System design MOC]]: code generation is the first deep dive of the URL shortener design.
- [[A PostgreSQL sequence never reuses a value taken by a rolled-back transaction, so identity keys have gaps]]:
  the counter alternative; a sequence-backed code never collides but has gaps, which are acceptable
  for a code.
- [[SELECT FOR UPDATE cannot prevent a double booking because there is no row to lock yet]]: the same
  pattern, where a unique key arbitrates the race instead of a check followed by an insert.
- Written up in win-interview: backend/docs/system-design-walkthrough.md, section 3.4
