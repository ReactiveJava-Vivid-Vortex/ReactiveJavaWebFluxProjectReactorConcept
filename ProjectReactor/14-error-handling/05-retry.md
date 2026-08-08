# retry()

## In Simple Terms

`.retry(n)` just tries again from the start, up to `n` times, whenever the
source fails — like retrying a phone call that dropped. If every attempt
still fails, it finally gives up and lets the last error through.

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

**Watch out for this:** plain `.retry(n)` tries again immediately, with no
pause in between — for anything talking to the real world (like a flaky
external service), you almost always want `.retryWhen()` with a proper
backoff instead.

## Why It Matters

`.retry()` is a simple fix for quick, transient hiccups (a momentary
network blip), but hammering a struggling service with no delay between
retries can actually make things worse — that's exactly why
`.retryWhen()` exists, for smarter retry behavior.
