# Flux.create()

## In Simple Terms

`Flux.create()` gives you a `FluxSink` you can use to emit items **from any thread, at
any time**, including from external, asynchronous, event-driven sources (like a
message listener, a WebSocket handler, or a hardware sensor callback). Unlike
`Flux.generate()`, it's not restricted to one-at-a-time synchronous emission — you can
push multiple items per callback invocation, from multiple threads even (with the
right sink configuration).

## Simple Example

```java
Flux<String> eventFlux = Flux.create(sink -> {
    ExternalEventSource source = new ExternalEventSource();

    source.onEvent(event -> sink.next(event));      // called from ANY thread
    source.onError(error -> sink.error(error));
    source.onClose(() -> sink.complete());

    sink.onDispose(() -> source.stopListening()); // cleanup when cancelled
});

eventFlux.subscribe(event -> System.out.println("Received event: " + event));
```

Because `Flux.create()` doesn't automatically pace emission to match downstream
demand, you should configure an overflow strategy for when the producer is faster
than the consumer:

```java
Flux.create(sink -> { /* ... */ }, FluxSink.OverflowStrategy.BUFFER);
```

## Why It Matters

`Flux.create()` is the primary bridge for integrating **push-based, external event
sources** (message queues, WebSockets, sensor data, legacy async callback APIs) into
a reactive pipeline. It's more flexible (and more dangerous if misused — e.g.,
unbounded buffering) than `Flux.generate()`, so it's best reserved for genuinely
asynchronous, multi-threaded sources.
