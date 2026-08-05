# take()

## In Simple Terms

`.take(n)` lets through only the **first `n` items** from a `Flux`, then automatically
cancels the upstream subscription and completes. There's also a `.take(Duration)`
overload that takes items for a fixed amount of time instead of a fixed count.

## Simple Example

```java
Flux.range(1, 100)
    .take(3)
    .subscribe(n -> System.out.println("Got: " + n));
```

Output:
```
Got: 1
Got: 2
Got: 3
```

Notice that even though the source has 100 items, `.take(3)` stops (and cancels the
upstream) after just 3 — the remaining 97 items are never even produced if the
source respects backpressure.

Time-based variant:

```java
Flux.interval(Duration.ofMillis(100))
    .take(Duration.ofSeconds(1)) // take items for 1 second, however many that is
    .subscribe(tick -> System.out.println("Tick: " + tick));
```

## Why It Matters

`.take()` is essential for bounding otherwise infinite or very large streams (e.g.,
`Flux.interval()`), and it's the standard way to write concise tests or demos without
manually cancelling subscriptions.
