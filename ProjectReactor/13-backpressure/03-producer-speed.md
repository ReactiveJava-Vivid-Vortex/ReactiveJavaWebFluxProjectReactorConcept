# Producer Speed

## In Simple Terms

"Producer speed" refers to how quickly a publisher can generate new items. When a
producer is much faster than its consumer, backpressure exists specifically to
prevent that speed mismatch from causing unbounded memory growth — the producer must
be told, and must respect, how much the consumer can currently handle.

## Simple Example

```java
Flux<Integer> fastProducer = Flux.range(1, 1_000_000); // can produce instantly

fastProducer
    .subscribe(new BaseSubscriber<Integer>() {
        @Override
        protected void hookOnSubscribe(Subscription subscription) {
            request(5); // artificially slow ourselves down as the consumer
        }

        @Override
        protected void hookOnNext(Integer value) {
            try { Thread.sleep(100); } catch (InterruptedException ignored) {} // slow processing
            System.out.println("Processed: " + value);
            request(1);
        }
    });
```

Even though the source `Flux.range()` could produce a million items nearly
instantly, the subscriber's controlled `request()` calls keep it in check — the
source only produces items as fast as they're requested.

## Why It Matters

A fast producer paired with a slow consumer, **without** backpressure, would cause
unbounded buffering (and eventual `OutOfMemoryError`). Reactive Streams' demand model
ensures producer speed is always tempered by actual consumer capacity — this is the
central problem backpressure exists to solve.
