# share()

## In Simple Terms

`.share()` turns a cold publisher into a hot one by having everyone
subscribed piggyback on a single, shared connection to the source. The
source kicks off when the first subscriber shows up, and shuts down once
the last one leaves — anyone who joins later only sees what happens *after*
they showed up, like walking into a live radio broadcast partway through.

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

Without `.share()`, every `.subscribe()` call would kick off a totally
separate `Flux.interval()` — its own independent clock ticking just for
that one subscriber.

## Why It Matters

`.share()` is the easiest way to have several consumers share one
expensive or naturally "live" source — a WebSocket connection, a sensor
feed — instead of each consumer accidentally spinning up its own redundant
copy of the same underlying work.
