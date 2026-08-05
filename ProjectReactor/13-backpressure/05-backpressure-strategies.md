# Backpressure Strategies

## In Simple Terms

Beyond raw overflow strategies on `Flux.create()`, Project Reactor provides dedicated
operators to apply backpressure-related behavior directly within a pipeline, letting
you decide how to react when a downstream consumer can't keep up.

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

Picking the right backpressure strategy is a real architectural decision tied to your
data's semantics — losing a stale sensor reading is fine, but silently dropping a
financial transaction is not. Reactor gives you explicit tools to make that trade-off
deliberately, rather than by accident.
