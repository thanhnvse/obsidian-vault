---
tags: [java, java-core, collections, interview]
status: draft
author: claude
source: "https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ArrayList.html"
created: 2026-09-30
score: 0.864
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# ArrayList is the default List because LinkedList walks its nodes on every indexed access

## Core idea
`ArrayList` is a resizable-array implementation of `List`: `get`, `set` and `size` run in
constant time, and `add` at the end runs in amortized constant time. `LinkedList` is a
doubly-linked list, and every operation that takes an index traverses the list from the
beginning or the end, whichever is closer, so a `for (int i...)` loop calling `get(i)` on a
`LinkedList` is quadratic. The `ArrayList` Javadoc adds that its constant factor is low compared
to that of `LinkedList`. In OpenJDK, `LinkedList` also allocates one `Node` object per element,
holding the item plus `next` and `prev` links, while `ArrayList` keeps each element in one slot
of an `Object[]`.

## Why choose / why not
- Choose `ArrayList` when: the list is built and then iterated or read by index, which covers
  almost every query result, DTO list and batch in a backend service.
- Choose `LinkedList` only when: you insert or remove at the position of a `ListIterator` you
  already hold, many times per pass, and a profile shows that shifting `ArrayList` elements is
  the cost; the unlink is constant time, but reaching the position is not.
- Don't use `LinkedList` as a queue or a stack: `ArrayDeque` is the array-backed choice for
  both.

## Interview angle
- Probed as "when would you pick `LinkedList` over `ArrayList`?"; the interviewer wants to hear
  that inserting in the middle is constant time only after a linear walk to the node.
- Common wrong answer: "`LinkedList` is faster for inserts and deletes."
- Strong answer: default to `ArrayList`, quote the constant-factor remark in its Javadoc, and
  say you would measure the real workload before switching.

## Related
- [[Java collections MOC]]: list choice is the opening collections question in the Java
  core section of the map, so this note is where that cluster starts.
