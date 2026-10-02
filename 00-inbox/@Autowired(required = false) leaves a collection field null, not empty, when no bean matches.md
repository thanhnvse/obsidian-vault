---
tags: [java, spring, dependency-injection, interview]
status: draft
author: claude
up: ["[[Spring MOC]]"]
source: "https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/autowired.html"
created: 2026-10-01
score: 0.897
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 2
---
# @Autowired(required = false) leaves a collection field null, not empty, when no bean matches

## Core idea
`@Autowired(required = false)` on a collection field avoids the startup failure that a required
collection field causes when no bean matches, because Spring expects at least one matching bean
for an array, collection or map injection point. With `required = false`, Spring then does not
populate the field at all and leaves its default value in place, so a `List<T>` field declared
without an initialiser stays `null` rather than holding an empty list. An empty collection is what a
class with a single constructor gets instead: there, a collection or map parameter with no matching
bean is resolved to an empty instance. A lab on Spring Framework 6.1.2 with no bean of the element
type confirmed this: the non-required field was `null`, and the single constructor received an
empty `List`.

## Why choose / why not
- Use constructor injection for plug-in lists that may be empty, such as checks that are all behind
  `@ConditionalOnProperty`: a single constructor gets an empty list, so no null check is needed.
- Initialise the field yourself, such as `List<OrderCheck> checks = List.of();`, when field
  injection must stay: Spring leaves the default in place, so the default is the empty list.
- Don't put `required = false` on a collection field unless every use checks for `null`: otherwise
  the first loop over it throws `NullPointerException`, and only in the environment that has no
  implementations.

## Interview angle
- Probed as "what is injected into a `List<T>` field marked `required = false` when there is no
  bean of type `T`?".
- Common wrong answer: "an empty list."
- Strong answer: nothing is injected, so the field keeps its default, `null`; only a single
  constructor turns a missing collection into an empty one, which is one more reason to prefer
  constructor injection.

## Related
- [[Injecting a List of an interface gives every bean of that type, sorted by @Order]]: partial
  overlap; that note gives the required failure and the single-constructor empty list, and this one
  adds the `required = false` field, which gets `null` instead.
