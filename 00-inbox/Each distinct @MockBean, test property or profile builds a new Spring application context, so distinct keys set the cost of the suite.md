---
tags: [java, spring, spring-boot, testing, interview]
status: draft
author: claude
up: ["[[Spring MOC]]"]
source: "https://docs.spring.io/spring/reference/6.2/testing/testcontext-framework/ctx-management/caching.html"
created: 2026-10-07
review: unjudged
---
# Each distinct @MockBean, test property or profile builds a new Spring application context, so distinct keys set the cost of the suite

## Core idea
Spring's TestContext framework builds an `ApplicationContext` for a test class and caches it under a
key made of the configuration classes, context customizers, test properties, active profiles, web
environment and context loader; a test class with the same key gets the same object. A `@MockBean`,
`properties` on `@SpringBootTest` or `@TestPropertySource`, `@ActiveProfiles`, a
`@DynamicPropertySource` or a different web environment each change the key, so Spring builds a new
context and creates every bean again, starting whatever those beans start, such as a connection pool
or a web server. The cache holds 32 contexts by default and evicts the least recently used one; the lab
asserts the 32 on Spring Framework 6.1.2, and the eviction rule comes from the cited 6.2 reference
page, not from a test. In a large suite the number of distinct keys, not the number of tests, is
the cost.

## Why choose / why not
- Share one base configuration for integration tests when: many classes need mocks; put the mocks in
  a `@TestConfiguration` that every test imports instead of a different `@MockBean` set per class,
  and set properties from one `@DynamicPropertySource` in a shared base class.
- Accept a new context when: only a few tests really need a different mock set or profile.
- Don't use `@DirtiesContext` as a cleanup habit: it closes the cached context, so the next class
  builds a new one; reset the state the test changed instead.
- Remember that shared contexts share mutable beans: state a test leaves in a shared bean is visible
  to the next test class, which is a flaky-test cause.
- On Boot 3.2.1 and Spring Framework 6.1.2 the annotation is `@MockBean`; `@MockitoBean` does not
  exist there and is documented from Spring Framework 6.2. The cache behaves the same for both.

## Interview angle
- Probed as "why is our test suite slow?".
- Common wrong answers: "`@MockBean` is free", and "use `@DirtiesContext` to clean up".
- Strong answer: count the application contexts (DEBUG on `org.springframework.test.context.cache`
  shows hits and misses), say the cache holds 32 by default, and fix it by sharing configuration;
  "the suite builds N contexts" beats "the suite is slow".

## Related
- [[Spring MOC]]: the parent map; test configuration decides how many containers a suite starts.
- [[A @Transactional test never commits, so it hides flush-time constraint errors, AFTER_COMMIT listeners and lazy-loading failures]]:
  the other cost of Spring test setup: this note is about speed, that one is about what a green test
  can prove.

Written up in win-interview: backend/java/docs/testing-strategy.md, section 2.5
