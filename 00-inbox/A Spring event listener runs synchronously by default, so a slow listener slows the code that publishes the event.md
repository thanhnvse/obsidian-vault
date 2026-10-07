---
tags: [java, spring, events, observer, interview]
status: draft
author: claude
up: ["[[Spring MOC]]"]
source: "https://docs.spring.io/spring-framework/reference/core/beans/context-introduction.html#context-functionality-events"
created: 2026-10-07
review: unjudged
---
# A Spring event listener runs synchronously by default, so a slow listener slows the code that publishes the event

## Core idea
`publishEvent(...)` calls each `@EventListener` method on the publisher's own thread. The Spring
reference says that by default listeners receive events synchronously, so `publishEvent()` blocks
until all of them have finished, and that a listener runs inside the publisher's transaction when
there is one. A lab on Spring Framework 6.1.2 adds three consequences: listeners run in `@Order`
order, lowest value first; a listener that throws reaches the publisher, and the later listeners
never run unless the multicaster has an `ErrorHandler`; and `@Async` on a listener, or an executor
on the multicaster, moves delivery to another thread, where the publisher no longer waits. The
reference adds that an exception from an asynchronous listener is not propagated to the caller. A
listener on another thread also runs outside the publisher's transaction. Decoupled in code is
therefore not asynchronous at run time.

## Why choose / why not
- Use an event when: the publisher should not know who reacts, such as cache eviction, an audit
  trail or plug-in modules.
- Call the collaborators directly when: there are one or two known ones; an event hides the control
  flow, and reading `placeOrder` you cannot see what runs when the event is published.
- Add `@Async` or a multicaster executor deliberately when: a slow listener, for example an email
  on an order event that made checkout slower, must not hold the publisher; then decide how failures
  are handled, because exceptions no longer reach the caller.
- Don't use an event for an effect that must not be lost: events are in-process, so a crash between
  the commit and the listener loses the effect; use an outbox.

## Interview angle
- Probed as "how do Spring application events work, and are they asynchronous?"
- Common wrong answer: "application events are asynchronous."
- Strong answer: synchronous on the publisher's thread by default, in `@Order` order, in the
  publisher's transaction, with a listener's exception reaching the publisher; `@Async` or an
  executor on the multicaster changes the thread, the transaction and where failures go; and being
  in-process, events need an outbox for effects that must not be lost.

## Related
- [[Spring MOC]]: the event publisher is the container's own observer mechanism, so delivery rules
  belong with the other container behaviour there.
- [[@TransactionalEventListener moves a side effect after the commit but loses it if the process dies]]:
  the variant that waits for the commit; it shares the in-process loss named here.
- [[Work handed to another thread runs outside the caller's Spring transaction]]: what an `@Async`
  listener loses, the publisher's transaction.
- Written up in win-interview: backend/java/docs/design-patterns.md, section 8.1
