# bufferTimeout()

## In Simple Terms

`.bufferTimeout(maxSize, maxTime)` batches items like `.buffer(n)`, but with an
additional time-based cutoff: it emits a batch as soon as **either** `maxSize` items
have accumulated, **or** `maxTime` has elapsed since the batch started — whichever
happens first. This prevents slow-trickling streams from waiting forever to fill a
full-size batch.

## Simple Example

```java
Flux.interval(Duration.ofMillis(100))
    .bufferTimeout(5, Duration.ofMillis(300))
    .subscribe(batch -> System.out.println("Batch: " + batch));
```

Since items arrive every 100ms, but the timeout is 300ms, you'll get partial batches
(around 3 items) rather than waiting to accumulate the full 5:

```
Batch: [0, 1, 2]
Batch: [3, 4, 5]
Batch: [6, 7, 8]
...
```

## Why It Matters

`.bufferTimeout()` solves a real problem with plain `.buffer(n)`: if items arrive
slowly or irregularly, a size-only buffer might never fill up, delaying processing
indefinitely. This is essential for near-real-time batching — e.g., batching log
events for shipping to a monitoring system, where you want batches quickly even if
they're not perfectly full.
