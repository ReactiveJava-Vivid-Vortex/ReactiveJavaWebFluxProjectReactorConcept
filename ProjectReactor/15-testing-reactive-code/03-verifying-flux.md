# Verifying Flux

## In Simple Terms

Testing a `Flux` with `StepVerifier` typically involves asserting the sequence and
count of emitted items, in addition to how the stream terminates.

## Simple Example

```java
@Test
void testFluxSequence() {
    Flux<Integer> flux = Flux.range(1, 5);

    StepVerifier.create(flux)
        .expectNext(1, 2, 3, 4, 5)
        .verifyComplete();
}
```

Asserting just the count, without checking every individual value:

```java
@Test
void testFluxCount() {
    Flux<Integer> flux = Flux.range(1, 100);

    StepVerifier.create(flux)
        .expectNextCount(100)
        .verifyComplete();
}
```

Testing with a custom predicate on each item:

```java
@Test
void testFluxWithPredicate() {
    Flux<Integer> flux = Flux.range(1, 5);

    StepVerifier.create(flux)
        .expectNextMatches(n -> n == 1)
        .expectNextMatches(n -> n == 2)
        .thenConsumeWhile(n -> n < 5) // consume the rest until this becomes false
        .verifyComplete();
}
```

## Why It Matters

Being able to assert both exact sequences and looser conditions (counts, predicates)
gives you the flexibility to write precise tests for small, deterministic streams,
and more resilient tests for large or loosely-specified ones — without your tests
becoming brittle to minor, irrelevant changes.
