# Flux.interval()

## In Simple Terms

`Flux.interval(duration)` creates a `Flux<Long>` that counts up (`0`, `1`, `2`,
...) on a fixed schedule, **forever** — it never finishes on its own. It's often
used to simulate a heartbeat, a polling loop, or a ticking clock.

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

A real-world use — powering a live-updating endpoint:

```java
@GetMapping(value = "/live-clock", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<String> liveClock() {
    return Flux.interval(Duration.ofSeconds(1))
        .map(tick -> LocalTime.now().toString());
}
```

**Heads up:** `Flux.interval()` runs on a background scheduler by default, not
the thread that subscribed — so tests using it usually need
`StepVerifier.withVirtualTime()` to avoid literally waiting in real time.

## Why It Matters

`Flux.interval()` is the standard building block for anything periodic in a
reactive app — heartbeats, live dashboards, health checks — without needing a
separate `Timer` or `ScheduledExecutorService`.
