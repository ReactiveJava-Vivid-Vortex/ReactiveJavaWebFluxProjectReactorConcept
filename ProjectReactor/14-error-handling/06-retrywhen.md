# retryWhen()

## In Simple Terms

`.retryWhen()` gives you fine control over how retries work — how many
times to try, how long to wait between attempts (including gradually
increasing the wait, known as backoff), and which errors are even worth
retrying in the first place. Reactor ships a handy `Retry` builder for
common patterns like this.

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

A simpler fixed-delay version (no gradually increasing wait):

```java
Mono.fromCallable(() -> callFlakyService())
    .retryWhen(Retry.fixedDelay(3, Duration.ofSeconds(1)))
    .subscribe();
```

## Why It Matters

`.retryWhen()` is the real-world tool for handling flaky external systems —
gradually increasing the wait between tries avoids piling more pressure on
a service that's already struggling, and filtering by error type makes sure
you're only retrying things worth retrying (there's no point retrying a
`400 Bad Request` — it'll never succeed, no matter how many times you try).
