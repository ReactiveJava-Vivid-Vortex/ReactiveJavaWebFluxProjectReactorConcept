# bufferTimeout()

## In Simple Terms

`.bufferTimeout()` batches items the same way `.buffer(n)` does, but adds a
safety net: it sends off whatever batch it has so far either when it fills
up, *or* when a time limit runs out — whichever happens first. That way, a
slow trickle of items doesn't leave you waiting forever for a full batch
that might never come.

## Simple Example

```java
Flux.interval(Duration.ofMillis(100))
    .bufferTimeout(5, Duration.ofMillis(300))
    .subscribe(batch -> System.out.println("Batch: " + batch));
```

Since items arrive every 100ms but the timeout is 300ms, you'll get
partial batches (around 3 items) rather than waiting to fill up all 5:

```
Batch: [0, 1, 2]
Batch: [3, 4, 5]
Batch: [6, 7, 8]
...
```

## Why It Matters

`.bufferTimeout()` fixes a real problem with plain `.buffer(n)`: if items
trickle in slowly or unevenly, a size-only batch might never fill up,
leaving your data stuck waiting. This matters a lot for near-real-time
batching — like grouping log events to ship off to a monitoring system,
where you'd rather send a partial batch quickly than wait indefinitely for
a full one.
