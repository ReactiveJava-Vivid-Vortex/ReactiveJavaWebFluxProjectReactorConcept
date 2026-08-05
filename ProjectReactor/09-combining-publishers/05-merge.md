# merge()

## In Simple Terms

`Flux.merge(source1, source2, ...)` combines multiple publishers **concurrently** —
it subscribes to all sources at once, and emits items from whichever source produces
them first. Unlike `concat()`, the order of items **interleaves** based on timing,
not on source order.

## Simple Example

```java
Flux<String> fast = Flux.just("A1", "A2").delayElements(Duration.ofMillis(100));
Flux<String> slow = Flux.just("B1", "B2").delayElements(Duration.ofMillis(150));

Flux.merge(fast, slow)
    .subscribe(item -> System.out.println("Got: " + item));
```

Output (interleaved based on actual timing, not always the same every run):
```
Got: A1
Got: B1
Got: A2
Got: B2
```

## concat() vs merge()

| Aspect      | concat()                         | merge()                              |
|-------------|-----------------------------------|----------------------------------------|
| Subscription| Sequential (one at a time)         | Concurrent (all sources at once)      |
| Ordering    | Strictly preserved                 | Interleaved, based on timing          |
| Speed       | Slower (waits for each to finish)  | Faster (all sources work in parallel) |

## Why It Matters

`.merge()` is ideal when you have multiple independent sources (e.g., calling several
downstream microservices) and you don't care about the order results arrive in —
just that you get them all as fast as possible, combining whichever finishes first.
