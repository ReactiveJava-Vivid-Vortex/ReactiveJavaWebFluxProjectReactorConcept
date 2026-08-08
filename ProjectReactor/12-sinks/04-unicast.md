# Unicast

## In Simple Terms

A unicast sink is built for exactly one listener at a time — like a private
phone line instead of a broadcast. It buffers whatever comes in until that
one subscriber is ready to receive it. Try to add a second listener at the
same time, and you'll typically get an error.

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

Unlike multicast, this single subscriber sees everything sent so far —
unicast sinks (with the `onBackpressureBuffer()` variant) hold onto items
until someone actually shows up to receive them.

## Why It Matters

Unicast sinks fit naturally when there's only ever going to be one consumer
for a stream — like a background task feeding results into exactly one
processing pipeline — and you want the safety of knowing (and enforcing)
that only one subscriber will ever attach, instead of using a general-
purpose multicast sink that technically allows many.
