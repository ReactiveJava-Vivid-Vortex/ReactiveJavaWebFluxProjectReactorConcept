# Unicast

## In Simple Terms

A **unicast** sink supports **only a single subscriber** at a time — it's designed
for the case where exactly one consumer will process the stream (buffering emitted
items until that one subscriber is ready). Attempting a second concurrent
subscription typically results in an error.

## Simple Example

```java
Sinks.Many<Integer> sink = Sinks.many().unicast().onBackpressureBuffer();
Flux<Integer> flux = sink.asFlux();

// Emit before anyone subscribes - buffered internally
sink.tryEmitNext(1);
sink.tryEmitNext(2);

flux.subscribe(v -> System.out.println("The one subscriber sees: " + v));
// Output:
// The one subscriber sees: 1
// The one subscriber sees: 2
```

Unlike multicast, this single subscriber sees **everything** emitted so far, since
unicast sinks buffer until a subscriber shows up (with the `onBackpressureBuffer()`
variant).

## Why It Matters

Unicast sinks are the right fit when there's naturally only one consumer for a
stream — e.g., a background task feeding results to exactly one processing pipeline
— and you want the safety net of a compile-time-enforced single-subscriber contract,
rather than a general-purpose multicast sink that could technically support many.
