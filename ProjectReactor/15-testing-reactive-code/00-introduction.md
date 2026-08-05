# Testing Reactive Code — Topic Overview

## What Is This Topic About? (In Simple Terms)

Testing asynchronous code with plain JUnit assertions is awkward — by the time your
assertion runs, has the async operation even finished? `StepVerifier` (from the
`reactor-test` module) solves this by subscribing to your `Mono`/`Flux` and letting
you declare **exactly** what should happen, step by step, blocking the test thread
in a controlled way until each expectation is satisfied (or a timeout fails the
test).

```java
StepVerifier.create(Flux.just(1, 2, 3))
    .expectNext(1, 2, 3)
    .verifyComplete();
```

Because a `Mono` has exactly three possible outcomes (value, empty, error — see the
Mono topic), your tests should cover all three, not just the happy path:

```java
StepVerifier.create(Mono.empty()).verifyComplete();          // empty case
StepVerifier.create(Mono.error(new RuntimeException("x")))
    .expectErrorMessage("x").verify();                        // error case
```

The trickiest — and most valuable — skill here is testing **time-based** operators
(`Flux.interval()`, retries with backoff) without your test suite actually sitting
there waiting for real seconds or hours to pass. `StepVerifier.withVirtualTime()`
fast-forwards a simulated clock instead, turning what would be a 3-hour test into
one that completes in milliseconds.

## Quick Revision Cheat Sheet

| # | Concept | One-Line Summary |
|---|---|---|
| 1 | **StepVerifier** | The core tool: subscribe to a Mono/Flux and declare expected signals step by step. |
| 2 | **Verifying Mono** | Test all 3 possible outcomes explicitly: value (`expectNext`), empty (`verifyComplete` alone), error. |
| 3 | **Verifying Flux** | Assert exact sequences (`expectNext(...)`), counts (`expectNextCount`), or predicates (`expectNextMatches`). |
| 4 | **Virtual time** | `.withVirtualTime()` fast-forwards a simulated clock — test hours-long timers in milliseconds. |
| 5 | **Error testing** | Assert exact exception type/message/predicate with `expectError()`/`expectErrorMessage()`/`expectErrorMatches()`. |
| 6 | **Completion testing** | Verify HOW a stream ends: `verifyComplete()`, `thenCancel()`, or bounded waits with `expectNoEvent()`. |

## How It All Fits Together

```
StepVerifier.create(mono_or_flux)
      │
      ├── .expectNext(...) / .expectNextCount(n) / .expectNextMatches(...)   ← assert VALUES
      │
      ├── .expectNoEvent(duration) / .thenAwait(duration)                   ← control TIME
      │      (wrap the source in withVirtualTime() to avoid real waiting)
      │
      └── .verifyComplete() / .expectError(...) / .thenCancel().verify()    ← assert the ENDING
```

Treat `StepVerifier` as non-negotiable for any non-trivial reactive method — it
turns "I think this works" into "I've proven exactly how this behaves in all three
outcome cases, including edge cases around timing and errors."
