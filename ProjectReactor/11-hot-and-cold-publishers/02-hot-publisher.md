# Hot Publisher

## In Simple Terms

A hot publisher keeps producing data whether anyone's listening or not, and
everyone who's currently subscribed shares the exact same live feed. It's
like tuning into live TV — you only see what's playing from the moment you
turn it on, not from the start of the broadcast.

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

Notice items `1` and `2` went out before any subscriber showed up — anyone
who joins later just misses them, unless a replay mechanism is in place.

## Why It Matters

Hot publishers are the right fit for live, shared state or events — a
stock price ticker, or a broadcast where every connected client should see
the exact same real-time data — instead of each client accidentally
triggering its own separate copy of the work.
