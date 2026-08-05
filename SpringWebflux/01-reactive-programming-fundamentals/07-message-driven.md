# Message Driven

## In Simple Terms

"Message Driven" is the foundational trait of the Reactive Manifesto: components
communicate through **asynchronous messages/events** rather than direct, synchronous
calls that block the caller. This loose coupling is what makes the other three
traits (Responsive, Resilient, Elastic) achievable.

## Simple Example

Direct, synchronous (tightly-coupled) communication:

```java
// Caller is blocked until the callee finishes - tight coupling in time
Result result = otherService.doSomething(request);
```

Message-driven (asynchronous) communication:

```java
// Caller isn't blocked - it reacts to the result whenever it arrives
otherService.doSomethingReactive(request)
    .subscribe(result -> handleResult(result));
```

In Spring WebFlux, every request/response is internally handled this way — `Mono`s
and `Flux`es represent asynchronous message streams flowing between the HTTP layer
and your application code, rather than direct blocking calls.

## Why It Matters

Message-driven architecture decouples components in **time** — the sender doesn't
need the receiver to be immediately available or fast, since communication happens
via asynchronous signals rather than a direct, blocking call-and-wait. This
decoupling is what allows reactive systems to isolate failures (resilience), absorb
load spikes (elasticity), and remain responsive under varying conditions.
