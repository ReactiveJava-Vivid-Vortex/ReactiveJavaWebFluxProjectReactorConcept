# timeout()

## In Simple Terms

`.timeout(duration)` makes a `Mono`/`Flux` fail with a `TimeoutException` if it
doesn't emit anything (or complete) within the given time window. It's essential for
avoiding indefinitely-hanging operations — e.g., a downstream service that never
responds.

## Simple Example

```java
Mono<String> slowCall = Mono.delay(Duration.ofSeconds(5)).map(t -> "Finally done");

slowCall
    .timeout(Duration.ofSeconds(2)) // fail if it takes longer than 2 seconds
    .subscribe(
        result -> System.out.println("Result: " + result),
        error -> System.out.println("Timed out: " + error.getMessage())
    );
```

Output (after 2 seconds, not 5):
```
Timed out: Did not observe any item or terminal signal within 2000ms
```

Combining `.timeout()` with a fallback:

```java
slowCall
    .timeout(Duration.ofSeconds(2))
    .onErrorResume(TimeoutException.class, e -> Mono.just("Fallback: took too long"))
    .subscribe(System.out::println);
```

## Why It Matters

`.timeout()` is a critical resilience tool for any call to an external system —
without it, a single unresponsive downstream dependency could cause requests to hang
indefinitely, eventually exhausting resources (like connections or threads) across
your entire application.
