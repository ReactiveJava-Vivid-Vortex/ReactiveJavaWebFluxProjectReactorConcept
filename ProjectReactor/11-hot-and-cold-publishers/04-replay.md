# replay()

## In Simple Terms

`.replay()` turns a cold publisher into a hot one that remembers some or
all of what it already sent, and plays that history back for any new
subscriber who joins later — unlike `.share()`, which only shows late
joiners what happens from that point forward, with nothing from before.

## Simple Example

```java
ConnectableFlux<Long> replayed = Flux.interval(Duration.ofSeconds(1))
    .replay(3); // remember the last 3 items for new subscribers

replayed.connect(); // start the underlying source immediately

try { Thread.sleep(3500); } catch (Exception e) {}

replayed.subscribe(t -> System.out.println("Late subscriber saw: " + t));
```

Output (the late subscriber immediately gets the last 3 cached items, then
keeps going live):
```
Late subscriber saw: 0
Late subscriber saw: 1
Late subscriber saw: 2
Late subscriber saw: 3   (live, continuing)
```

You can also replay the *entire* history (`replay()` with no arguments, or
`.cache()` for a simpler version meant for streams that eventually finish).

## Why It Matters

`.replay()` is perfect when new subscribers need a bit of context, not just
"whatever happens from now on" — a chat room showing the last few messages
to anyone who joins, or a dashboard replaying recent metrics to a newly
connected viewer.
