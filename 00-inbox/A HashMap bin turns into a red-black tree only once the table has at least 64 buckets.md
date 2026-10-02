---
tags: [java, java-core, collections, interview]
status: draft
author: claude
up: ["[[Java collections MOC]]"]
source: "https://github.com/openjdk/jdk21u/blob/master/src/java.base/share/classes/java/util/HashMap.java"
created: 2026-09-30
score: 0.847
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A HashMap bin turns into a red-black tree only once the table has at least 64 buckets

## Core idea
Since JDK 8, `HashMap` can store a crowded bin as a red-black tree of `TreeNode`s instead of a
linked list of `Node`s. In the OpenJDK 21 source, `putVal` calls `treeifyBin` when a put adds to a
bin that already holds `TREEIFY_THRESHOLD` (8) nodes, and `treeifyBin` resizes the table instead of
building a tree while the table has fewer than `MIN_TREEIFY_CAPACITY` (64) buckets. With keys that
all share one hash code, a 16-bucket `HashMap` therefore resizes to 32 buckets on the 9th key, to 64
buckets on the 10th key, and converts the bin into a tree on the 11th key. A tree bin that a resize
splits down to `UNTREEIFY_THRESHOLD` (6) nodes becomes a list again. A tree bin orders its nodes by
hash and then by `compareTo` when the keys are `Comparable`, which keeps a lookup in a crowded bin
at O(log n).

## Why choose / why not
- Count on tree bins when: keys come from outside and may collide, by accident or on purpose; they
  are the defence JEP 180 added, and they work best when the key type is `Comparable`.
- Don't design for them when: you control the key type; with a well-spread `hashCode` the source's
  own estimate for a bin of 8 is about 0.00000006, so trees almost never form. Fix a poor `hashCode`
  instead.
- Don't expect help when: colliding keys are not `Comparable`; the tree cannot order them and may
  have to search both subtrees.

## Interview angle
- Probed as "what changed in `HashMap` in Java 8?".
- Common wrong answer: "a bucket becomes a tree at 8 entries", which misses the 64-bucket condition
  and the resize that happens first.
- Strong answer: give the three constants (8, 6, 64), the resize-first rule, and why the trees
  exist: O(log n) instead of O(n) under heavy collisions, for `Comparable` keys.

## Related
- [[Overriding equals without hashCode makes HashMap lookups miss]]: both follow from the bin walk,
  where the hash picks the bucket before `equals` (or `compareTo` in a tree bin) picks the node.
- [[TreeMap trades HashMap's constant time for sorted keys and range queries]]: the source says tree
  bins are structured like a `TreeMap`, but only inside one crowded bucket, not for the whole map.
