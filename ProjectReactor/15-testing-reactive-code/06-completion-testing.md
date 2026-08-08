# Completion Testing

## In Simple Terms

Checking *how* a stream ends — cleanly, with an error, or by being
cancelled — matters just as much as checking what values it produced along
the way. `StepVerifier` gives you a separate assertion for each of those
endings.

## Simple Example

Checking a clean finish:

```java
@Test
void testCompletion() {
    Flux<Integer> flux = Flux.just(1, 2, 3);

    StepVerifier.create(flux)
        .expectNextCount(3)
        .verifyComplete(); // asserts onComplete() specifically
}
```

Checking that a stream deliberately does *not* finish within a time budget
(useful for confirming an infinite stream really is unbounded elsewhere):

```java
@Test
void testTimeout() {
    Flux<Long> flux = Flux.never(); // never emits, never completes

    StepVerifier.create(flux)
        .expectSubscription()
        .expectNoEvent(Duration.ofMillis(100))
        .thenCancel() // explicitly cancel instead of waiting forever
        .verify();
}
```

Checking cancellation behavior:

```java
@Test
void testCancellation() {
    Flux<Integer> flux = Flux.range(1, 100);

    StepVerifier.create(flux)
        .expectNext(1)
        .thenCancel()
        .verify();
}
```

## Why It Matters

Explicitly checking how a stream ends catches subtle bugs that a test only
looking at emitted values would completely miss — like a pipeline that
should finish but instead hangs forever, or one that's supposed to error
out but silently completes as if nothing went wrong.
