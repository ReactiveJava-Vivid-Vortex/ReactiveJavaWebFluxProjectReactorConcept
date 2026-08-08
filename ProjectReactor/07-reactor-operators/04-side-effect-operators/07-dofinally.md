# doFinally()

## In Simple Terms

`.doFinally()` runs no matter how the stream ends — whether it finishes
cleanly, fails with an error, or gets cancelled partway through. It tells
you exactly which of the three happened. It's the reactive version of a
`finally` block from regular try/catch/finally code — it always runs, no
excuses.

## Simple Example

```java
Flux.just(1, 2, 3)
    .doFinally(signalType -> System.out.println("Finished with signal: " + signalType))
    .subscribe(n -> System.out.println("Item: " + n));
```

Output:
```
Item: 1
Item: 2
Item: 3
Finished with signal: onComplete
```

Guaranteed cleanup, even on error or cancellation:

```java
Flux.interval(Duration.ofSeconds(1))
    .doFinally(signal -> System.out.println("Cleanup ran, reason: " + signal))
    .take(3) // "onComplete"
    .subscribe();
```

The signal you get back can be `ON_COMPLETE`, `ON_ERROR`, or `CANCEL`.

## Why It Matters

`.doFinally()` is the right place for cleanup you absolutely cannot skip —
releasing a resource, closing a connection, bringing down an "active tasks"
counter — regardless of whether the stream succeeded, blew up, or got cut
off early (like a client disconnecting mid-request).
