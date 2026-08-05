# Verifying Mono

## In Simple Terms

Testing a `Mono` with `StepVerifier` follows the same three possible outcomes a
`Mono` can have: a value, empty, or an error — your test asserts which one occurred.

## Simple Example

Testing a `Mono` that emits a value:

```java
@Test
void testMonoWithValue() {
    Mono<String> mono = Mono.just("Hello");

    StepVerifier.create(mono)
        .expectNext("Hello")
        .verifyComplete();
}
```

Testing an empty `Mono`:

```java
@Test
void testEmptyMono() {
    Mono<String> mono = Mono.empty();

    StepVerifier.create(mono)
        .verifyComplete(); // no expectNext() call - nothing was emitted
}
```

Testing a `Mono` that errors:

```java
@Test
void testMonoError() {
    Mono<String> mono = Mono.error(new IllegalArgumentException("bad input"));

    StepVerifier.create(mono)
        .expectErrorMatches(error ->
            error instanceof IllegalArgumentException &&
            error.getMessage().equals("bad input")
        )
        .verify();
}
```

## Why It Matters

Explicitly testing all three possible `Mono` outcomes (value, empty, error) ensures
your service methods behave correctly in every scenario — not just the "happy path"
— catching regressions where, say, an empty case accidentally starts throwing an
exception, or vice versa.
