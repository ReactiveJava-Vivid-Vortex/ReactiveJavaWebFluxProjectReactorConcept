# StepVerifier

## In Simple Terms

`StepVerifier` is the standard way to test `Mono`/`Flux` code. It
subscribes to your stream and lets you spell out exactly what you expect,
step by step — what values should come out, in what order, and how it
should all wrap up.

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

Without `StepVerifier`, testing async code with plain JUnit assertions is
awkward — race conditions, manually juggling `CountDownLatch`es. `StepVerifier`
subscribes and waits in a controlled, predictable way until the expected
things happen (or times out), making reactive tests just as easy to write
as regular synchronous ones.
