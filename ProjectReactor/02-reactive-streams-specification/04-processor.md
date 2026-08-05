# Processor

## In Simple Terms

A `Processor` is both a `Subscriber` **and** a `Publisher` at the same time. It sits
in the *middle* of a pipeline: it subscribes to an upstream source (consuming data),
optionally transforms it, and re-publishes it downstream to its own subscribers.

```java
public interface Processor<T, R> extends Subscriber<T>, Publisher<R> {
}
```

## Simple Example

Think of a processor like a relay station: it receives a signal, maybe amplifies or
converts it, and re-broadcasts it.

```
Publisher(raw sensor data) -> Processor(converts Celsius to Fahrenheit) -> Subscriber
```

In modern Project Reactor code, you rarely implement `Processor` directly anymore —
it has largely been **replaced by `Sinks`** (covered in a later section), which offer
a safer, easier-to-use API for the same "bridge" role.

```java
// Modern replacement using Sinks instead of a raw Processor:
Sinks.Many<Integer> sink = Sinks.many().multicast().onBackpressureBuffer();
sink.tryEmitNext(1);
sink.asFlux().subscribe(System.out::println);
```

## Why It Matters

Understanding `Processor` explains historically how you'd manually bridge
"push data in, push transformed data out" before Reactor introduced `Sinks`. Knowing
this helps you understand *why* `Sinks` exist and what problem they solve.
