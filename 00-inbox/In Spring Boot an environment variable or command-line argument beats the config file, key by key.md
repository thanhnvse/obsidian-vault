---
tags: [java, spring, spring-boot, configuration, interview]
status: draft
author: claude
up: ["[[Spring MOC]]"]
source: "https://docs.spring.io/spring-boot/docs/3.2.1/reference/html/features.html#features.external-config"
created: 2026-10-07
review: unjudged
---
# In Spring Boot an environment variable or command-line argument beats the config file, key by key

## Core idea
Spring Boot looks a key up in an ordered list of property sources, and the first source that has it
answers. The part that matters day to day, highest precedence first, is: command-line arguments,
`SPRING_APPLICATION_JSON`, Java system properties, OS environment variables, `random.*`, config
files (a profile-specific file above the base file), then default properties. A higher source hides
only the keys it sets; every other key still resolves from a lower source. In a lab on Boot 3.2.1 a
system property beat the file and a command-line argument beat both, and an OS environment variable
beat the file while losing to a system property. A placeholder such as `lab.url=${lab.host}:5432` is
resolved against the winning value, so `--lab.host=other` gives `other:5432`.

## Why choose / why not
- Override one value in production with an environment variable or a command-line argument rather
  than editing the jar's file; it wins without touching the other keys.
- Don't assume the file wins: a stray (lạc, ngoài ý muốn) environment variable on a host with the
  same name as one of your keys beats the file on that machine only.
- When a value "will not change", list the property sources of the running environment before
  suspecting the file; the Actuator `env` endpoint shows where a value came from.
- Choose the active profiles from outside the artifact, with an environment variable or an argument,
  so one build runs everywhere; a profile file holds values that differ per environment.

## Interview angle
- Probed as "which value wins when the same property is set in several places?", and as "it works on
  my machine but not on the server".
- Common wrong answer: "`application.properties` always wins."
- Strong answer: the precedence order, that sources are merged by key and not replaced, the
  practical rule (environment variable or argument over file), and how to ask the running
  application instead of guessing.

## Related
- [[Spring MOC]]: the parent map; configuration sources decide what value a bean receives.
- [[Declaring your own ObjectMapper bean makes Boot's back off and silently drops the spring.jackson properties]]:
  the other way a setting seems to do nothing: the value is found, but the bean it was meant for is
  no longer created.

Written up in win-interview: backend/java/docs/spring-boot-fundamentals.md, sections 2.5 and 2.6
