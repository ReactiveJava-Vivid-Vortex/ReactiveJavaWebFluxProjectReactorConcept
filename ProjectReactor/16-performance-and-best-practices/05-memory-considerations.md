# Memory Considerations

## In Simple Terms

Reactive pipelines can still eat up a lot of memory if you're careless —
certain operators (`.buffer()`, `.collectList()`, `.distinct()`, unbounded
`Sinks`) hold data in memory, and knowing which ones do (and roughly how
much) matters if you want to avoid trouble at scale.

## Simple Example

Operators that hold data in memory — use these with an eye on how big your
data actually is:

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

There's a common misconception that "reactive" automatically means
"memory-efficient." It doesn't — some everyday operators can quietly build
up unbounded state if you're not paying attention to which ones you're
using on large or endless streams. Knowing which operators hold data in
memory, and putting explicit bounds on them, matters a lot for keeping
production systems stable.
