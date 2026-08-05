# retry()

## In Simple Terms

`.retry(n)` automatically **re-subscribes** to the upstream `Mono`/`Flux` up to `n`
times if it errors, essentially "trying again from the start" on failure. If all `n`
retries also fail, the last error is finally propagated downstream.

## Simple Example

```java
AtomicInteger attempts = new AtomicInteger(0);

Mono.fromCallable(() -> {
    int attempt = attempts.incrementAndGet();
    System.out.println("Attempt #" + attempt);
    if (attempt < 3) {
        throw new RuntimeException("Simulated failure");
    }
    return "Success!";
})
.retry(2) // retry up to 2 additional times after the first failure
.subscribe(
    result -> System.out.println("Result: " + result),
    error -> System.out.println("All retries exhausted: " + error.getMessage())
);
```

Output:
```
Attempt #1
Attempt #2
Attempt #3
Result: Success!
```

**Important gotcha:** plain `.retry(n)` retries **immediately**, with no delay
between attempts — for real-world resilience (e.g., calling a flaky external
service), you almost always want `.retryWhen()` with a backoff strategy instead.

## Why It Matters

`.retry()` is a simple tool for transient, quickly-resolving failures (e.g., a
momentary network blip), but its lack of any delay between attempts can worsen load
on an already-struggling downstream service — which is exactly why `.retryWhen()`
exists for more sophisticated retry policies.
