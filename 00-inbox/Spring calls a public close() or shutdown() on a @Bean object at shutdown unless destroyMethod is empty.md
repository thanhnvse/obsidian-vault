---
tags: [java, spring, bean-lifecycle, interview]
status: draft
author: claude
up: ["[[Spring MOC]]"]
source: "https://docs.spring.io/spring-framework/docs/6.1.x/javadoc-api/org/springframework/context/annotation/Bean.html"
created: 2026-09-30
score: 0.877
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# Spring calls a public close() or shutdown() on a @Bean object at shutdown unless destroyMethod is empty

## Core idea
For an object returned from a `@Bean` method, the container infers a destroy method: it detects a
public, no-argument method named `close` or `shutdown` and calls it when the application context
closes. Detection is reflective against the bean instance at creation time, so it works whatever
return type the `@Bean` method declares. `@Bean(destroyMethod = "")` switches the inference off for
that bean; a `DisposableBean` callback is still invoked. Destroy methods run only for beans whose
lifecycle is under the full control of the factory, which is always the case for singletons.

## Why choose / why not
- Rely on it for resources the context created: a pool or executor built inside the `@Bean` method
  is closed at shutdown with no extra code.
- Set `destroyMethod = ""` for objects the context does not own, such as a pool or executor shared
  with other code or looked up elsewhere: otherwise the context shuts them down when it closes,
  which often shows up between integration tests that share the object.
- Don't expect it for prototype-scoped beans: their destruction is not under the factory's control.

## Interview angle
- Probed as "who closes the `DataSource` you declared with `@Bean`?".
- Common wrong answer: "nothing, unless it implements `DisposableBean`".
- Strong answer: destroy-method inference on `close`/`shutdown`, the empty-string opt-out, and the
  shared-resource case where the opt-out is needed.

## Related
- [[Spring does not call @PreDestroy on prototype-scoped beans]]: the same destruction phase seen
  from the scope side; inference does not help a prototype either, because the container keeps no
  record of it.
