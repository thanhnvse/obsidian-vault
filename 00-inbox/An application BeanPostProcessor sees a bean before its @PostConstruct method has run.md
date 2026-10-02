---
tags: [java, spring, bean-lifecycle, interview]
status: draft
author: claude
up: ["[[Spring MOC]]"]
source: "https://docs.spring.io/spring-framework/docs/6.1.x/javadoc-api/org/springframework/context/annotation/CommonAnnotationBeanPostProcessor.html"
created: 2026-10-01
score: 0.877
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# An application BeanPostProcessor sees a bean before its @PostConstruct method has run

## Core idea
`@PostConstruct` is not a separate step before the bean post-processors. Spring's
`CommonAnnotationBeanPostProcessor` supports it as an init annotation, through
`InitDestroyAnnotationBeanPostProcessor`, and invokes it from its `postProcessBeforeInitialization`
callback; the `BeanFactory` javadoc lists `postProcessBeforeInitialization` of the bean
post-processors as one step, before `afterPropertiesSet()` and the custom init method. When the
context registers its post-processors, it re-registers the internal ones, those that also
implement `MergedBeanDefinitionPostProcessor` such as `CommonAnnotationBeanPostProcessor`, at the
end of the list. An ordinary application `BeanPostProcessor` therefore receives the injected bean
in `postProcessBeforeInitialization` before its `@PostConstruct` method has run, and receives it
again in `postProcessAfterInitialization` after all init callbacks. A lab on Spring Framework 6.1.2
recorded this order.

## Why choose / why not
- Check state that `@PostConstruct` sets up in `postProcessAfterInitialization` when: the check
  depends on that state; in the before-init callback it does not exist yet.
- Use `postProcessBeforeInitialization` when: the post-processor must act on the injected but not
  yet initialised bean, such as supplying a value its init method reads.
- Don't expect an `@Order` or `PriorityOrdered` value to move an ordinary post-processor after
  `@PostConstruct`: the internal post-processors are re-registered last whatever the application's
  order values.

## Interview angle
- Probed as "where exactly does `@PostConstruct` run?" or "does a `BeanPostProcessor` see the bean
  before or after `@PostConstruct`?".
- Common wrong answer: "`@PostConstruct` runs before all `BeanPostProcessor`s."
- Strong answer: `@PostConstruct` is itself a before-init post-processor callback, Spring's
  post-processor for it runs last in that loop, so an application before-init hook sees the
  uninitialised object and the after-init hook sees the initialised bean or its proxy.

## Related
- [[@Transactional does not apply inside @PostConstruct because the proxy is created after initialisation]]:
  partial overlap; that note places `@PostConstruct` before `postProcessAfterInitialization`, where
  the proxy is created, while this one places it inside the before-init step, after the
  application's own post-processors.
- [[A BeanFactoryPostProcessor edits bean definitions before the container instantiates any other bean]]:
  the other extension point with a similar name; it changes definitions before any bean exists,
  while the post-processors in this note act on each bean instance.
