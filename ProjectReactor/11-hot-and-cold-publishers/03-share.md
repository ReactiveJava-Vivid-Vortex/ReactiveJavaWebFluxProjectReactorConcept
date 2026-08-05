# share()

## In Simple Terms

`.share()` converts a cold publisher into a hot one by having all subscribers share
a **single underlying subscription** to the source. The source starts executing when
the *first* subscriber arrives, and stops (unsubscribes from the source) when the
*last* subscriber leaves — new subscribers joining later only see items emitted
*after* they subscribed.

## Simple Example

```java
Flux<Long> shared = Flux.interval(Duration.ofSeconds(1))
    .share();

shared.subscribe(t -> System.out.println("Subscriber A: " + t));

try { Thread.sleep(2500); } catch (Exception e) {}

shared.subscribe(t -> System.out.println("Subscriber B: " + t)); // joins mid-stream
```

Output (Subscriber B misses ticks 0 and 1, joining only from tick 2 onward):
```
Subscriber A: 0
Subscriber A: 1
Subscriber A: 2
Subscriber B: 2
Subscriber A: 3
Subscriber B: 3
```

Without `.share()`, each `.subscribe()` call would trigger an entirely separate,
independent `Flux.interval()` execution — a completely different ticking clock for
each subscriber.

## Why It Matters

`.share()` is the simplest way to make an expensive or naturally "live" source (like
a WebSocket connection, or a sensor feed) shared across multiple consumers, instead of
each consumer accidentally triggering its own separate, redundant execution of the
same underlying resource.
