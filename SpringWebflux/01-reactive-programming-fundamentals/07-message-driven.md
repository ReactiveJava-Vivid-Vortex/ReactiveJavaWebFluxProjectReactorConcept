# Message Driven

## In Simple Terms

"Message Driven" is the foundational idea behind the Reactive Manifesto:
parts of a system talk to each other through async messages or events,
instead of direct calls that freeze the caller until they get an answer.
This loose coupling is what actually makes the other three traits
(Responsive, Resilient, Elastic) possible.

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

In Spring WebFlux, every request and response is handled this way
internally — `Mono`s and `Flux`es are async message streams flowing
between the HTTP layer and your code, instead of direct calls that freeze
and wait.

## Why It Matters

Message-driven architecture decouples parts of a system in *time* — the
sender doesn't need the receiver to be instantly available or fast, since
they're talking through async signals instead of a direct call-and-wait.
That decoupling is exactly what lets reactive systems keep failures
contained (resilience), absorb sudden spikes (elasticity), and stay
responsive no matter what's going on.
