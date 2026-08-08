# Replay (Sinks)

## In Simple Terms

A replay sink remembers some or all of what it already sent, and plays that
history back for any new subscriber who joins later — combining
multicast's "many listeners at once" with the "catch up on what happened"
ability you'd get from `.replay()` on a `Flux`.

## Simple Example

```java
Sinks.Many<String> sink = Sinks.many().replay().limit(2); // remember last 2 items
Flux<String> flux = sink.asFlux();

sink.tryEmitNext("Event 1");
sink.tryEmitNext("Event 2");
sink.tryEmitNext("Event 3");

// A late subscriber still catches the last 2 events:
flux.subscribe(e -> System.out.println("Late subscriber: " + e));
```

Output:
```
Late subscriber: Event 2
Late subscriber: Event 3
```

You can also use `.replay().all()` to remember everything since the sink
was first created.

## Why It Matters

Replay sinks are great for cases where new subscribers need instant
context — a chat app showing the last few messages to anyone who just
joined a room, or a monitoring dashboard immediately showing recent history
to a new client instead of leaving them staring at a blank screen until the
next live update.
