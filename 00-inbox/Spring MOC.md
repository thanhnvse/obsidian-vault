---
tags: [moc, java, spring, interview]
type: moc
status: draft
author: claude
up: ["[[Java backend interview MOC]]"]
created: 2026-09-30
---
# Spring MOC

The question behind this map: *what does the container do to my bean, and when?*

## Transactions
- [[@Transactional MOC]]: rollback rules, propagation and the proxy limits interviewers probe

## Injection and lifecycle
- [[Injecting a List of an interface gives every bean of that type, sorted by @Order]]: collection injection
- [[Injecting a String-keyed Map gives a strategy registry keyed by bean name]]: the strategy pattern without if/else
- [[@Transactional does not apply inside @PostConstruct because the proxy is created after initialisation]]: where the lifecycle meets AOP
- [[Spring does not call @PreDestroy on prototype-scoped beans]]: the lifecycle gap for prototypes

## HTTP clients
- [[RestClient replaces RestTemplate as Spring's synchronous HTTP client]]: which client new code should use
- [[WebClient saves threads only when the caller does not block on it]]: when reactive pays off
- [[RestClient and WebClient take their timeouts from the underlying HTTP library]]: where timeouts are actually configured
