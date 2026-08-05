# StepVerifier

## In Simple Terms

`StepVerifier` (from the `reactor-test` module) is the standard tool for testing
`Mono`/`Flux` pipelines. It lets you subscribe to a reactive stream and declare
**exactly what you expect to happen**, step by step — which values should be emitted,
in what order, and how the stream should end (complete, error, etc.).

## Simple Example

```java
@Test
void testSimpleFlux() {
    Flux<Integer> flux = Flux.just(1, 2, 3);

    StepVerifier.create(flux)
        .expectNext(1)
        .expectNext(2)
        .expectNext(3)
        .verifyComplete();
}
```

Testing transformations:

```java
@Test
void testMap() {
    Flux<String> flux = Flux.just(1, 2, 3).map(n -> "Item-" + n);

    StepVerifier.create(flux)
        .expectNext("Item-1", "Item-2", "Item-3")
        .verifyComplete();
}
```

## Why It Matters

Without `StepVerifier`, testing async/reactive code with plain JUnit assertions is
awkward and unreliable (race conditions, needing manual `CountDownLatch`es).
`StepVerifier` subscribes and blocks the test thread in a controlled, deterministic
way until the expected signals occur (or a timeout is hit), making reactive pipeline
tests as straightforward to write as synchronous ones.
