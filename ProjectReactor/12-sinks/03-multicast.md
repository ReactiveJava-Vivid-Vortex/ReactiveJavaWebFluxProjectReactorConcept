# Multicast

## In Simple Terms

A **multicast** sink broadcasts each emitted item to **all currently subscribed**
subscribers at once — like a live radio broadcast, everyone tuned in hears the same
thing at the same time. Subscribers who join late miss anything emitted before they
subscribed (unless combined with replay).

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

If a third subscriber joins after `tryEmitNext(1)` was called, it will **not** see
that value — only values emitted after it subscribed.

## Why It Matters

Multicast sinks are the right choice whenever multiple, independent subscribers all
need to see the **same live events** — e.g., a shared SSE broadcast to several
connected browser clients, or an internal event bus with multiple listeners all
reacting to the same events.
