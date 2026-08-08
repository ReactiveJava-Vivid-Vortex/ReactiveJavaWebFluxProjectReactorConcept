# Infinite Streams

## In Simple Terms

An **infinite stream** is a `Flux` that, by design, never calls `onComplete()` —
it just keeps sending items until something explicitly cancels it. Live sensor
feeds, stock tickers, and `Flux.interval()` are all classic examples.

## Simple Example

```java
Flux<Long> infiniteTicks = Flux.interval(Duration.ofMillis(500));

// If you subscribe without limiting it, this runs forever:
// infiniteTicks.subscribe(tick -> System.out.println(tick));

// Safer: limit it explicitly
infiniteTicks
    .take(Duration.ofSeconds(3)) // stop after 3 seconds of ticks
    .subscribe(tick -> System.out.println("Tick: " + tick));
```

Operators commonly used to tame infinite streams:

| Operator            | Purpose                                        |
|----------------------|-------------------------------------------------|
| `.take(n)`            | Stop after `n` items                            |
| `.take(Duration)`     | Stop after a fixed amount of time               |
| `.takeUntil(predicate)` | Stop once a condition becomes true             |
| Manual `.dispose()`   | Cancel from outside based on external logic     |

## Why It Matters

Infinite streams are great for real-time features (live prices, chat, live
updates), but they come with a risk: forget to bound them, and they can run
forever, quietly leaking resources. In Spring WebFlux, an infinite `Flux`
returned from a controller is automatically cancelled once the client
disconnects — but standalone code needs to manage this itself.
