# Flux.interval()

## In Simple Terms

`Flux.interval(duration)` creates a `Flux<Long>` that emits an incrementing counter
(`0`, `1`, `2`, ...) at a fixed time interval, **forever** (it never completes on its
own). It's commonly used to simulate periodic events, like a heartbeat, polling
mechanism, or ticking clock.

## Simple Example

```java
Flux.interval(Duration.ofSeconds(1))
    .take(5) // limit to 5 ticks, otherwise it runs forever
    .subscribe(tick -> System.out.println("Tick: " + tick));
```

Output (one line per second):
```
Tick: 0
Tick: 1
Tick: 2
Tick: 3
Tick: 4
```

A realistic use case — powering a Server-Sent Events endpoint with periodic updates:

```java
@GetMapping(value = "/live-clock", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<String> liveClock() {
    return Flux.interval(Duration.ofSeconds(1))
        .map(tick -> LocalTime.now().toString());
}
```

**Important:** `Flux.interval()` runs by default on a parallel `Scheduler`, not the
calling thread — so tests using it typically need `StepVerifier.withVirtualTime()` to
avoid actually waiting in real time.

## Why It Matters

`Flux.interval()` is the go-to building block for any periodic/polling behavior in a
reactive application — heartbeats, live dashboards, scheduled health checks — all
without needing a separate `Timer` or `ScheduledExecutorService`.
