# replay()

## In Simple Terms

`.replay()` converts a cold publisher into a hot, **connectable** one that remembers
(caches) some or all past emitted items, and replays them to any **new** subscriber
who joins later — unlike `.share()`, which only shows new subscribers what happens
*after* they join.

## Simple Example

```java
ConnectableFlux<Long> replayed = Flux.interval(Duration.ofSeconds(1))
    .replay(3); // remember the last 3 items for new subscribers

replayed.connect(); // start the underlying source immediately

try { Thread.sleep(3500); } catch (Exception e) {}

replayed.subscribe(t -> System.out.println("Late subscriber saw: " + t));
```

Output (the late subscriber immediately receives the last 3 cached items, then
continues live):
```
Late subscriber saw: 0
Late subscriber saw: 1
Late subscriber saw: 2
Late subscriber saw: 3   (live, continuing)
```

You can also replay **all** history (`replay()` with no arguments, or `.cache()` for
a simpler variant meant for finite, completing sources).

## Why It Matters

`.replay()` is perfect when new subscribers need some historical context, not just
"whatever happens from now on" — e.g., a chat room showing the last N messages to
anyone who joins, or a monitoring dashboard replaying recent metric history to newly
connected viewers.
