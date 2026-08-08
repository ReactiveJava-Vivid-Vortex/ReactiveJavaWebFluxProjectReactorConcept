# Verifying Flux

## In Simple Terms

Testing a `Flux` with `StepVerifier` usually means checking the sequence and
count of items it emits, plus how it finally ends.

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

Just asserting the count, without checking every single value:

```java
@Test
void testFluxCount() {
    Flux<Integer> flux = Flux.range(1, 100);

    StepVerifier.create(flux)
        .expectNextCount(100)
        .verifyComplete();
}
```

Testing with a custom check on each item:

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

Being able to check both exact sequences and looser conditions (counts,
custom checks) gives you the flexibility to write tight tests for small,
predictable streams, and looser, more forgiving tests for big or
loosely-defined ones — without your tests breaking over tiny, irrelevant
changes.
