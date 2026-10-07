---
tags: [protobuf, grpc, schema-evolution, microservices, interview]
status: draft
author: claude
up: ["[[Microservices and messaging MOC]]"]
source: "https://protobuf.dev/programming-guides/proto3/"
created: 2026-10-07
review: unjudged
---
# Reusing a protobuf field number for another field of the same wire type makes old data decode as the new field, silently

## Core idea
The protobuf wire format carries field numbers and wire types, not field names, so a rename changes
no bytes. A number reused for another field of the same type is therefore misread, not rejected: in
the write-up's test an old writer's `quantity` of 5 was read as `priceInCents` 5, with no error.
With a different wire type the value goes to the unknown fields and the new field stays empty, and
changing a field's number loses the value the same way, so the reader sees 0. The proto3 guide calls
decoding after a reuse ambiguous (mơ hồ), with outcomes that include data corruption, and the
best-practice page says never to re-use a tag number and to reserve the number of a deleted field.
The write-up built its schemas in code and ran no `protoc`, so it did not test whether the compiler
enforces `reserved`.

## Why choose / why not
- Add a field with a new number when: extending a message; an old reader parses it and keeps the new
  field as unknown.
- Reserve a deleted field's number and name when: removing a field; old bytes can still be in flight
  or stored.
- Don't renumber because "both sides are updated": during a rollout both versions exist.
- Don't change a type outside a compatible group: `int32`, `int64` and `bool` are compatible, but
  `int32` to `sint32` is not, because `sint32` is zig-zag encoded and 5 reads as -3.
- Forward a message instead of rebuilding it: copying only the known fields into a new message
  drops the new ones.

## Interview angle
- Probed as "how do you change a protobuf message without breaking consumers?"
- Common wrong answers: "renumbering is fine if we update both sides", or "`int32` to `sint32` is
  harmless, both are integers".
- Strong answer: add with new numbers, never renumber or reuse, reserve deleted numbers, treat a
  missing field as the default, use `optional` where "not set" matters, deploy readers before
  writers, and say that the failures are silent wrong values, not errors.

## Related
- [[Microservices and messaging MOC]]: independent deployment depends on messages that both the old
  and the new version can read, which is what these rules protect.
- [[Expand and contract schema changes keep the previous version runnable after a rollback]]: the same
  rule at a database boundary; while a rollout runs, old and new versions coexist, so a change must
  be readable by both.
- Written up in win-interview: backend/docs/microservice-infrastructure.md, section 2.7
