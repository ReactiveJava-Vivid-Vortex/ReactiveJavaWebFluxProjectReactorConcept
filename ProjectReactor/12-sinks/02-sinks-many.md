# Sinks.Many

## In Simple Terms

`Sinks.Many<T>` is the modern way to manually produce a **stream of multiple values**
(like a programmatic `Flux`), replacing older `Processor` implementations
(`DirectProcessor`, `EmitterProcessor`, etc.). You build one with a specific
distribution strategy (multicast, unicast, or replay), then call `tryEmitNext()`
whenever you have a new item to push.

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

`Sinks.Many` is the modern building block for any "manually push events into a
reactive stream" scenario — internal event buses, bridging message queues, or
broadcasting live updates (e.g., SSE endpoints) — replacing the older, harder-to-use
`Processor` API entirely.
