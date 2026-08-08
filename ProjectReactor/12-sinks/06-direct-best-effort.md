# Direct Best Effort

## In Simple Terms

Some sink setups (like `Sinks.many().multicast().directBestEffort()`) don't
buffer anything at all — if a subscriber isn't ready to receive right that
second, the item is simply dropped for them, on a "best effort" basis,
rather than being held in memory until they catch up.

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

If a subscriber is slow to ask for more items, some emissions may come back
as a failure (like `FAIL_OVERFLOW`) instead of quietly piling up in memory
forever.

## Why It Matters

"Direct, best effort" behavior makes sense when it's actually fine to lose
some data under pressure — like broadcasting live sensor readings, where
the freshest value matters more than never missing a single one, or a
live-updating UI where skipping an intermediate frame is harmless as long
as the latest state eventually shows up. You're trading guaranteed delivery
for keeping memory usage bounded.
