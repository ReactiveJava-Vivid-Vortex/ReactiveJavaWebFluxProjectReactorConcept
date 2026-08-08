# Error Testing

## In Simple Terms

Testing what happens when things go wrong matters just as much as testing
the happy path. `StepVerifier` gives you several ways to check that a
`Mono`/`Flux` fails exactly the way you expect it to.

## Simple Example

Checking the exact exception type:

```java
@Test
void testErrorType() {
    Mono<Integer> mono = Mono.error(new IllegalStateException("invalid state"));

    StepVerifier.create(mono)
        .expectError(IllegalStateException.class)
        .verify();
}
```

Checking the exception type *and* message:

```java
@Test
void testErrorMessage() {
    Mono<Integer> mono = Mono.error(new IllegalStateException("invalid state"));

    StepVerifier.create(mono)
        .expectErrorMessage("invalid state")
        .verify();
}
```

Checking with a custom rule on the error:

```java
@Test
void testErrorMatches() {
    Mono<Integer> mono = Mono.error(new CustomException("E123", "Something failed"));

    StepVerifier.create(mono)
        .expectErrorMatches(error ->
            error instanceof CustomException &&
            ((CustomException) error).getCode().equals("E123")
        )
        .verify();
}
```

Checking that some values come through *before* the error hits:

```java
@Test
void testPartialSuccessBeforeError() {
    Flux<Integer> flux = Flux.just(1, 2, 0, 4).map(n -> 10 / n);

    StepVerifier.create(flux)
        .expectNext(10, 5)
        .expectError(ArithmeticException.class)
        .verify();
}
```

## Why It Matters

Testing errors explicitly makes sure your error-handling logic
(`onErrorResume`, `retryWhen`, custom exception mapping) is actually
proven to work, not just assumed to. This is a common gap — happy-path
tests pass fine, and then production traffic reveals error-handling bugs
that nobody ever checked for.
