# Flux.create()

## In Simple Terms

`Flux.create()` hands you a `FluxSink` that you can use to push items into a
stream **from anywhere, at any time** — including from an outside, async source
like a message listener, a WebSocket, or a sensor callback. Unlike
`Flux.generate()`, you're not limited to one item at a time — you can push
several items per callback, even from different threads.

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

Because `Flux.create()` doesn't automatically slow down to match demand, you
should tell it what to do if the producer gets ahead of the consumer:

```java
Flux.create(sink -> { /* ... */ }, FluxSink.OverflowStrategy.BUFFER);
```

## Why It Matters

`Flux.create()` is the main way to plug **push-based, outside event sources**
(message queues, WebSockets, sensors, legacy callback APIs) into a reactive
pipeline. It's more flexible than `Flux.generate()` — but also easier to misuse
(unbounded buffering, for instance) — so save it for genuinely async,
multi-threaded sources.
