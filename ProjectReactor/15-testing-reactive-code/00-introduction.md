# Q1. Why Can't I Just Test Reactive Code with Plain JUnit Assertions?

## Simple Explanation (Think of Ordering Food vs Getting It Instantly)

```java
User user = fetchUser(); // synchronous — by the time this line finishes, "user" is ready
assertEquals("Alice", user.getName()); // safe to assert immediately

Mono<User> userMono = fetchUserReactive(); // returns INSTANTLY — nothing has happened yet!
assertEquals("Alice", userMono.???); // there's no value here to assert on YET
```

You can't just "read the result" of a `Mono`/`Flux` the way you read a return
value — you have to **subscribe and wait, in a controlled way**, for the async
result to actually arrive. `StepVerifier` does exactly that for you.

---

## Q2. What Does a Basic `StepVerifier` Test Look Like?

```java
@Test
void testSimpleFlux() {
    StepVerifier.create(Flux.just(1, 2, 3))
        .expectNext(1)
        .expectNext(2)
        .expectNext(3)
        .verifyComplete();
}
```

It subscribes to the `Flux`, blocks the test thread until each expectation is
satisfied (or a timeout fails the test), and asserts the exact sequence of
signals.

---

## Q3. How Do I Test All Three `Mono` Outcomes?

```java
// Success
StepVerifier.create(Mono.just("Hello")).expectNext("Hello").verifyComplete();

// Empty — NOTE: no expectNext() call at all
StepVerifier.create(Mono.empty()).verifyComplete();

// Error
StepVerifier.create(Mono.error(new IllegalArgumentException("bad")))
    .expectErrorMatches(e -> e instanceof IllegalArgumentException
        && e.getMessage().equals("bad"))
    .verify();
```

**Test all three** — not just the happy path. This is the single biggest gap in
poorly-tested reactive code.

---

## Q4. How Do I Test `Flux.interval()` Without Waiting 3 Real Hours?

```java
@Test
void testIntervalWithVirtualTime() {
    StepVerifier.withVirtualTime(() -> Flux.interval(Duration.ofHours(1)).take(3))
        .expectSubscription()
        .expectNoEvent(Duration.ofHours(1))
        .expectNext(0L)
        .thenAwait(Duration.ofHours(1))
        .expectNext(1L)
        .thenAwait(Duration.ofHours(1))
        .expectNext(2L)
        .verifyComplete();
}
```

`.withVirtualTime()` fast-forwards a **simulated** clock instead of the test
actually sleeping for hours. **Critical gotcha:** the `Flux`/`Mono` must be
created **inside** the lambda passed to `withVirtualTime()` — creating it
beforehand means the virtual clock isn't installed in time.

---

## Q5. How Do I Assert an Exact Sequence vs a Loose Condition?

```java
StepVerifier.create(Flux.range(1, 5))
    .expectNext(1, 2, 3, 4, 5)      // exact sequence
    .verifyComplete();

StepVerifier.create(Flux.range(1, 100))
    .expectNextCount(100)            // just the count, don't care about exact values
    .verifyComplete();

StepVerifier.create(Flux.range(1, 5))
    .expectNextMatches(n -> n == 1)  // predicate-based, per item
    .thenConsumeWhile(n -> n < 5)
    .verifyComplete();
```

---

## Q6. How Do I Test Cancellation?

```java
StepVerifier.create(Flux.range(1, 100))
    .expectNext(1)
    .thenCancel()   // cancel instead of waiting for completion
    .verify();
```

---

## Q7. Interview-Style Q&A

### Does `StepVerifier.create(mono)` automatically subscribe?

Building the verifier doesn't subscribe by itself — calling a terminal method like
`.verify()`, `.verifyComplete()`, or `.verifyError()` triggers the actual
subscription and blocks until the expected signals occur.

### Why would a `StepVerifier` test hang forever?

If you assert `.verifyComplete()` on a `Flux` that never actually completes (e.g.
`Flux.interval()` without `.take()`), the test will hang until it times out and
fails — always bound infinite sources in tests.

### What's the difference between `expectNext()` and `expectNextMatches()`?

`expectNext()` checks exact equality; `expectNextMatches()` takes a predicate,
useful when you only care about some property of the value, not its exact
identity.

---

## Q8. Summary

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

### One sentence to remember

> **"You can't read a Mono's value like a variable — StepVerifier subscribes
> for you and waits, in a controlled way, for exactly the signals you
> expect."**
