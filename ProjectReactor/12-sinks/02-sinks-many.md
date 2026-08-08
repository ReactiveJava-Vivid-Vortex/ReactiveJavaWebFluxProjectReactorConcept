# Sinks.Many

## In Simple Terms

`Sinks.Many<T>` is the modern way to manually push out a stream of multiple
values — a `Flux` you control by hand, replacing older tools like
`DirectProcessor` and `EmitterProcessor`. You build one with a specific
"who gets what" strategy (multicast, unicast, or replay), then call
`tryEmitNext()` every time you have something new to send out.

## Simple Example

```java
Sinks.Many<String> sink = Sinks.many().multicast().onBackpressureBuffer();

Flux<String> flux = sink.asFlux();
flux.subscribe(value -> System.out.println("Subscriber got: " + value));

sink.tryEmitNext("Event 1");
sink.tryEmitNext("Event 2");
sink.tryEmitComplete();
```

Output:
```
Subscriber got: Event 1
Subscriber got: Event 2
```

## Why It Matters

`Sinks.Many` is the go-to building block for any "manually push events into
a stream" situation — an internal event bus, bridging a message queue, or
broadcasting live updates (like SSE endpoints) — replacing the old, harder
to use `Processor` API entirely.
