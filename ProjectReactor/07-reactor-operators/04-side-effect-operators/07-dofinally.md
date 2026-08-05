# doFinally()

## In Simple Terms

`.doFinally(consumer)` runs a side effect **no matter how the stream ends** —
successfully (`onComplete`), with an error (`onError`), or via cancellation. It
receives a `SignalType` telling you exactly which of the three happened. It's the
reactive equivalent of a `finally` block in traditional try/catch/finally.

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

Possible `SignalType` values include `ON_COMPLETE`, `ON_ERROR`, and `CANCEL`.

## Why It Matters

`.doFinally()` is the go-to place for **guaranteed cleanup logic** — releasing a
resource, closing a connection, decrementing an "active tasks" metric — that must
run regardless of whether the stream succeeded, failed, or was cancelled midway
(e.g., because a client disconnected).
