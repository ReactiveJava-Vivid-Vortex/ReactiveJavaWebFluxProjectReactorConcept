# Error Testing

## In Simple Terms

Testing error scenarios is just as important as testing the happy path in reactive
code — `StepVerifier` provides several ways to assert that a `Mono`/`Flux` fails in
exactly the way you expect.

## Simple Example

Asserting the exact exception type:

```java
@Test
void testErrorType() {
    Mono<Integer> mono = Mono.error(new IllegalStateException("invalid state"));

    StepVerifier.create(mono)
        .expectError(IllegalStateException.class)
        .verify();
}
```

Asserting the exception type AND message:

```java
@Test
void testErrorMessage() {
    Mono<Integer> mono = Mono.error(new IllegalStateException("invalid state"));

    StepVerifier.create(mono)
        .expectErrorMessage("invalid state")
        .verify();
}
```

Asserting with a custom predicate on the error:

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

Testing that some values are emitted **before** the error occurs:

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

Explicit error testing ensures your error-handling logic (`onErrorResume`,
`retryWhen`, custom exception mapping) is actually verified by tests, not just
assumed to work — a common gap where "happy path" tests pass, but production
failures reveal untested error-handling bugs.
