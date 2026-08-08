# Backpressure Strategies

## In Simple Terms

Beyond the raw overflow strategies you set on `Flux.create()`, Reactor also
gives you dedicated operators you can drop right into a pipeline to decide
what happens when a consumer just can't keep up.

## Simple Example

```java
// onBackpressureBuffer(): queue excess items (with an optional max size + overflow action)
Flux.range(1, 1000)
    .onBackpressureBuffer(100, dropped -> System.out.println("Dropped: " + dropped))
    .subscribe(slowConsumer());

// onBackpressureDrop(): silently discard items that exceed demand
Flux.range(1, 1000)
    .onBackpressureDrop(dropped -> System.out.println("Dropped: " + dropped))
    .subscribe(slowConsumer());

// onBackpressureLatest(): keep only the most recent item when overloaded
Flux.range(1, 1000)
    .onBackpressureLatest()
    .subscribe(slowConsumer());

// onBackpressureError(): fail loudly instead of silently dropping/buffering
Flux.range(1, 1000)
    .onBackpressureError()
    .subscribe(slowConsumer());
```

## Choosing a Strategy

| Strategy               | Best for...                                                    |
|--------------------------|--------------------------------------------------------------------|
| `onBackpressureBuffer()`  | Bursty traffic that should eventually be processed, not lost       |
| `onBackpressureDrop()`    | High-frequency data where losing some items is acceptable          |
| `onBackpressureLatest()`  | Live state updates where only the newest value matters              |
| `onBackpressureError()`   | Situations where silent data loss is unacceptable — fail fast instead |

## Why It Matters

Picking the right strategy is a real design call tied to what your data
actually means — losing a stale sensor reading is no big deal, but silently
dropping a financial transaction absolutely is. Reactor gives you clear
tools to make that trade-off on purpose, instead of it happening to you by
accident.
