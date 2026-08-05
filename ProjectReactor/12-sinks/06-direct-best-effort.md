# Direct Best Effort

## In Simple Terms

Some sink configurations (like plain `Sinks.many().multicast().directBestEffort()` or
the underlying "direct" strategies) don't buffer at all — if a subscriber isn't
currently ready to receive (has no outstanding demand), the emission is simply
**dropped** for that subscriber, on a "best effort" basis, rather than being queued
up in memory.

## Simple Example

```java
Sinks.Many<Integer> sink = Sinks.many().multicast().directBestEffort();
Flux<Integer> flux = sink.asFlux();

flux.subscribe(v -> System.out.println("Fast subscriber: " + v));

for (int i = 1; i <= 5; i++) {
    Sinks.EmitResult result = sink.tryEmitNext(i);
    if (result.isFailure()) {
        System.out.println("Emission " + i + " failed: " + result);
    }
}
```

If a subscriber is slow to request more items, some emissions may report a failure
result (like `FAIL_OVERFLOW` or similar) instead of being buffered indefinitely.

## Why It Matters

"Direct, best effort" semantics are appropriate when **losing some data under
backpressure is acceptable** — e.g., broadcasting live sensor readings where the
absolute latest value matters more than never missing one, or a UI live-update
stream where dropping an intermediate frame is fine as long as the latest state
eventually arrives. This trades guaranteed delivery for bounded memory usage.
