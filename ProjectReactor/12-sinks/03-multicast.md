# Multicast

## In Simple Terms

A multicast sink sends every item out to *all* currently subscribed
listeners at once — like a live radio broadcast, everyone tuned in hears
the same thing at the same moment. Anyone who tunes in late just misses
whatever already aired (unless you pair it with replay).

## Simple Example

```java
Sinks.Many<Integer> sink = Sinks.many().multicast().onBackpressureBuffer();
Flux<Integer> flux = sink.asFlux();

flux.subscribe(v -> System.out.println("Subscriber A: " + v));
flux.subscribe(v -> System.out.println("Subscriber B: " + v));

sink.tryEmitNext(1); // BOTH A and B receive this
```

Output:
```
Subscriber A: 1
Subscriber B: 1
```

If a third subscriber joins after `tryEmitNext(1)` already fired, they
won't see that value — only whatever gets emitted after they subscribe.

## Why It Matters

Multicast sinks are the right pick when several independent subscribers all
need to see the exact same live events — a shared SSE broadcast to
multiple browser tabs, or an internal event bus with several listeners all
reacting to the same happenings.
