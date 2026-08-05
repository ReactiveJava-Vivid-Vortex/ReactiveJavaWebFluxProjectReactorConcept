# Hot Publisher

## In Simple Terms

A **hot publisher** produces its data **regardless of whether anyone is subscribed**,
and all current subscribers share the **same, single, ongoing execution** — like
tuning into a live TV broadcast: you only see what's being broadcast from the moment
you tune in, not from the beginning.

## Simple Example

```java
Sinks.Many<Integer> sink = Sinks.many().multicast().onBackpressureBuffer();
Flux<Integer> hot = sink.asFlux();

// Emitting starts independent of subscribers
sink.tryEmitNext(1);
sink.tryEmitNext(2);

hot.subscribe(v -> System.out.println("Subscriber A: " + v));

sink.tryEmitNext(3); // only Subscriber A sees this (and anything after they joined)

hot.subscribe(v -> System.out.println("Subscriber B: " + v));

sink.tryEmitNext(4); // both A and B see this
```

Notice items `1` and `2` were emitted before any subscriber joined — they're lost to
subscribers who arrive later (unless a replay mechanism is used).

## Why It Matters

Hot publishers are ideal for representing **live, shared state or events** — like a
live stock price ticker, or a shared SSE broadcast where all connected clients should
see the same real-time data — rather than each client triggering its own independent
data-fetching execution.
