# Verifying Mono

## In Simple Terms

Testing a `Mono` with `StepVerifier` just means checking which of its three
possible outcomes actually happened: a value, nothing at all, or an error.

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

Testing all three possible outcomes — not just the happy path — makes sure
your service methods behave correctly no matter what happens. It's how you
catch bugs like an empty case that accidentally starts throwing an
exception, or the other way around.
