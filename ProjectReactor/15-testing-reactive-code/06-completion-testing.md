# Completion Testing

## In Simple Terms

Verifying how a stream **ends** — successfully (`onComplete`), with an error
(`onError`), or via cancellation — is just as important as verifying its emitted
values. `StepVerifier` provides distinct terminal assertions for each case.

## Simple Example

Verifying successful completion:

```java
@Test
void testCompletion() {
    Flux<Integer> flux = Flux.just(1, 2, 3);

    StepVerifier.create(flux)
        .expectNextCount(3)
        .verifyComplete(); // asserts onComplete() specifically
}
```

Verifying the stream does NOT complete within a time budget (e.g., testing an
infinite stream is correctly bounded elsewhere):

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

Verifying cancellation behavior:

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

Explicitly asserting *how* a stream terminates catches subtle bugs — like a pipeline
that should complete but instead hangs indefinitely, or one that's supposed to
error but silently completes instead — issues that a test only checking emitted
values (and ignoring the terminal signal) would completely miss.
