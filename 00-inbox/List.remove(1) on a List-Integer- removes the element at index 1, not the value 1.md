---
tags: [java, java-core, collections, interview]
status: draft
author: claude
up: ["[[Java collections MOC]]"]
source: "https://docs.oracle.com/javase/specs/jls/se21/html/jls-15.html#jls-15.12.2"
created: 2026-10-01
score: 0.898
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# List.remove(1) on a List<Integer> removes the element at index 1, not the value 1

## Core idea
`List` has two `remove` overloads: `remove(int index)` removes the element at that position, and
`remove(Object o)` removes the first element equal to `o`. JLS §15.12.2 resolves overloads in phases,
and the first phase does not permit boxing or unboxing conversion. For `list.remove(1)` the argument
`1` is an `int`, so `remove(int)` is applicable in the first phase and is chosen before
`remove(Object)`, which would need boxing, is ever considered. On a `List<Integer>` holding
`[1, 2, 3]`, `remove(1)` therefore removes the element at index 1 and leaves `[1, 3]`, while
`remove(Integer.valueOf(1))` removes the value 1 and leaves `[2, 3]`. The compiler gives no error or
warning for either call.

## Why choose / why not
- Pass a boxed value, as in `list.remove(Integer.valueOf(id))`, when: you mean "remove this value"
  from a list of `Integer`; the `Object` overload is then chosen in the first phase, because
  `remove(int)` would need unboxing.
- Use `removeIf(x -> x.equals(id))` when: every occurrence must go, or the intent should be readable
  without knowing the overload rules.
- Keep `remove(int)` when: you really remove by position, such as dropping the first element; on a
  list whose elements are not integers there is no ambiguity.

## Interview angle
- Probed as a code-reading puzzle: "a `List<Integer>` holds `[1, 2, 3]`; what is left after
  `list.remove(1)`?"
- Common wrong answer: "`[2, 3]`, because the value 1 is removed."
- Strong answer: `[1, 3]`; phase one of overload resolution picks `remove(int)` without boxing, so
  the call removes index 1. Then show `remove(Integer.valueOf(1))` for the value.

## Related
- [[Overriding equals without hashCode makes HashMap lookups miss]]: the `Object` overload finds its
  target with `equals`, so removing by value depends on the same equality contract as a map lookup.
- [[ArrayList is the default List because LinkedList walks its nodes on every indexed access]]: on a
  `LinkedList` the index overload also walks the nodes to reach the position, which that note explains.
