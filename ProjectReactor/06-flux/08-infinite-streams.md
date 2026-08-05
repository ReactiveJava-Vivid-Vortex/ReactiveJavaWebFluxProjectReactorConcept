# Infinite Streams

## In Simple Terms

An **infinite stream** is a `Flux` that, by design, never calls `onComplete()` — it
just keeps emitting items indefinitely, until explicitly cancelled. Examples include
live sensor feeds, stock price tickers, or `Flux.interval()`.

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

Infinite streams are extremely useful for real-time features (live prices, chat
messages, SSE feeds), but they carry a risk: if you forget to bound them (via
`.take()`, cancellation on client disconnect, etc.), they can run forever and leak
resources. In Spring WebFlux, an infinite `Flux` returned from a controller is
automatically cancelled when the client disconnects — but standalone code needs
explicit management.
