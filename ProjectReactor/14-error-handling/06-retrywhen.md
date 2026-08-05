# retryWhen()

## In Simple Terms

`.retryWhen(retrySpec)` gives you full, fine-grained control over retry behavior —
how many times to retry, how long to wait between attempts (including exponential
backoff), and which errors are even worth retrying. Project Reactor provides a
built-in `Retry` builder (`reactor.util.retry.Retry`) for common patterns.

## Simple Example

```java
Mono.fromCallable(() -> callFlakyService())
    .retryWhen(
        Retry.backoff(3, Duration.ofMillis(100)) // up to 3 retries, exponential backoff starting at 100ms
            .maxBackoff(Duration.ofSeconds(2))
            .filter(error -> error instanceof TimeoutException) // only retry timeouts
            .onRetryExhaustedThrow((spec, signal) ->
                new RuntimeException("Retries exhausted after " + signal.totalRetries() + " attempts")
            )
    )
    .subscribe(
        result -> System.out.println("Result: " + result),
        error -> System.out.println("Failed permanently: " + error.getMessage())
    );
```

Simple fixed-delay retries (no exponential backoff):

```java
Mono.fromCallable(() -> callFlakyService())
    .retryWhen(Retry.fixedDelay(3, Duration.ofSeconds(1)))
    .subscribe();
```

## Why It Matters

`.retryWhen()` is the production-grade tool for handling transient failures against
real external systems — exponential backoff avoids hammering an already-struggling
service, and filtering by exception type ensures you only retry errors that are
actually worth retrying (e.g., not retrying a `400 Bad Request`, which will never
succeed no matter how many times you try).
