# Processor

## In Simple Terms

A `Processor` is both a `Subscriber` **and** a `Publisher` at the same time. It
sits in the middle of a pipeline: it listens to data coming in from one side,
maybe changes it, and passes it along to whoever's listening on the other side.

```java
public interface Processor<T, R> extends Subscriber<T>, Publisher<R> {
}
```

## Simple Example

Think of it like a relay station: it picks up a signal, maybe boosts or converts
it, and rebroadcasts it further.

```
Publisher(raw sensor data) -> Processor(converts Celsius to Fahrenheit) -> Subscriber
```

In modern Project Reactor code, you almost never build a `Processor` by hand
anymore — it's been mostly replaced by `Sinks` (covered later in this course),
which do the same job but are much harder to get wrong.

```java
// Modern replacement using Sinks instead of a raw Processor:
Sinks.Many<Integer> sink = Sinks.many().multicast().onBackpressureBuffer();
sink.tryEmitNext(1);
sink.asFlux().subscribe(System.out::println);
```

## Why It Matters

Knowing what a `Processor` is helps explain *why* `Sinks` exist — they solve the
exact same "receive data, push data back out" problem, just more safely.
