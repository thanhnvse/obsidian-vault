---
tags: [java, spring, bean-lifecycle, interview]
status: draft
author: claude
up: ["[[Spring MOC]]"]
source: "https://docs.spring.io/spring-framework/reference/core/beans/factory-extension.html"
created: 2026-10-01
score: 0.857
review: "ready"
score_reasons: ["numbers: verify every figure yourself"]
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A BeanFactoryPostProcessor edits bean definitions before the container instantiates any other bean

## Core idea
A `BeanFactoryPostProcessor` operates on bean configuration metadata: the container lets it read
the bean definitions and change them before it instantiates any beans other than
`BeanFactoryPostProcessor` instances. Spring itself uses this hook for work on definitions rather
than objects: `PropertySourcesPlaceholderConfigurer` resolves `${...}` placeholders in definitions,
and `ConfigurationClassPostProcessor`, a `BeanDefinitionRegistryPostProcessor` subtype, parses
`@Configuration` classes into definitions. Because no regular bean exists yet at that point, the
hook is meant for definitions only: working with bean instances inside it, for example through
`getBean()`, causes premature instantiation, and the Spring reference warns that such beans can
bypass bean post-processing. A lab on Spring Framework 6.1.2 confirmed that a recording
`BeanFactoryPostProcessor` ran before any bean's constructor.

## Why choose / why not
- Choose a `BeanFactoryPostProcessor` when: the change concerns how beans will be created, such as
  registering or removing definitions or changing a definition's class, scope or property values,
  before any regular bean exists.
- Don't choose it when: the change concerns the bean objects themselves, such as wrapping them in a
  proxy or checking an annotated instance; that is a `BeanPostProcessor`'s job, which receives each
  instance as it is initialised.
- Don't call `getBean()` from a `BeanFactoryPostProcessor`: the bean is instantiated prematurely,
  before the normal lifecycle, so it can miss auto-proxying and other bean post-processing.

## Interview angle
- Probed as "`BeanPostProcessor` vs `BeanFactoryPostProcessor`?".
- Common wrong answer: "both modify beans; one just runs a bit earlier."
- Strong answer: definitions versus instances, the order inside the context refresh, one example of
  each from Spring itself, and the premature-instantiation trap.

## Related
- [[@Transactional does not apply inside @PostConstruct because the proxy is created after initialisation]]:
  the `BeanPostProcessor` side of this split; the auto-proxy creator is a bean post-processor that
  wraps each instance after its init callbacks.
- [[An application BeanPostProcessor sees a bean before its @PostConstruct method has run]]: the
  instance-side extension point that this note contrasts with, and where in initialisation it runs.
