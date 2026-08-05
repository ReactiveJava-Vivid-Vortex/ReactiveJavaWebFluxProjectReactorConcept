# Replay (Sinks)

## In Simple Terms

A **replay** sink remembers some or all previously emitted items and replays them to
any **new** subscriber who joins later — combining the "many subscribers" nature of
multicast with the "catch up on history" ability of `.replay()` on a `Flux`.

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

You can also use `.replay().all()` to remember the entire history since the sink was
created.

## Why It Matters

Replay sinks are ideal for scenarios where new subscribers need immediate context —
e.g., a chat application showing the last few messages to anyone who just joined a
room, or a monitoring dashboard immediately showing recent history to a newly
connected client instead of a blank screen until the next live event arrives.
