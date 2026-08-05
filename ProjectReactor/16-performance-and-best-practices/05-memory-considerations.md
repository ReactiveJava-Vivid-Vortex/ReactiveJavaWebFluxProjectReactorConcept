# Memory Considerations

## In Simple Terms

Reactive pipelines can still consume significant memory if used carelessly — certain
operators (`.buffer()`, `.collectList()`, `.distinct()`, unbounded `Sinks`) hold data
in memory, and understanding which ones do (and how much) is important for avoiding
memory issues at scale.

## Simple Example

Operators that hold data in memory — use with awareness of dataset size:

```java
// Loads ALL items into memory before emitting anything
hugeFlux.collectList().subscribe(list -> process(list));

// .distinct() remembers every unique value seen so far - grows unboundedly
// with high-cardinality data
hugeFlux.distinct().subscribe();

// Unbounded onBackpressureBuffer() can grow without limit if the consumer
// never catches up
fastProducerFlux.onBackpressureBuffer().subscribe(slowConsumer());
```

Safer, bounded alternatives:

```java
// Process in bounded batches instead of collecting everything
hugeFlux.buffer(1000).flatMap(batch -> processBatch(batch)).subscribe();

// Bound the buffer explicitly, with an overflow strategy for excess
fastProducerFlux
    .onBackpressureBuffer(10_000, dropped -> log.warn("Dropped: {}", dropped))
    .subscribe(slowConsumer());
```

## Why It Matters

A common misconception is that "reactive = automatically memory-efficient." In
reality, certain common operators can silently accumulate unbounded state if you're
not careful about which ones you use with large or unbounded streams — knowing which
operators hold data in memory (and bounding them explicitly) is essential for
production reliability.
