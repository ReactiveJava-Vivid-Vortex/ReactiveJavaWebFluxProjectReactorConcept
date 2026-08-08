# Reactive Streaming APIs

## In Simple Terms

Building a solid SSE-based API in WebFlux usually means combining a few
pieces covered elsewhere: a `Sinks.Many` to broadcast events, a `Flux`
returned from a controller with `produces = TEXT_EVENT_STREAM_VALUE`, and
often named events using `ServerSentEvent<T>` for more structured
client-side handling.

## Simple Example

Using `ServerSentEvent` for named events with IDs (which supports
resuming automatically after a reconnect):

```java
@GetMapping(value = "/events", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<ServerSentEvent<String>> streamEvents() {
    return Flux.interval(Duration.ofSeconds(1))
        .map(tick -> ServerSentEvent.<String>builder()
            .id(String.valueOf(tick))
            .event("tick-event")
            .data("Tick number " + tick)
            .build());
}
```

Backing it with a `Sinks.Many` for broadcasting real application events
(not just a timer):

```java
@Service
public class NotificationService {
    private final Sinks.Many<String> sink = Sinks.many().multicast().onBackpressureBuffer();

    public void notify(String message) {
        sink.tryEmitNext(message);
    }

    public Flux<ServerSentEvent<String>> subscribe() {
        return sink.asFlux()
            .map(msg -> ServerSentEvent.<String>builder().data(msg).build());
    }
}
```

## Why It Matters

Combining `Sinks` (to produce events), `ServerSentEvent` (for structured,
resumable events), and a streaming media type gives you a complete,
production-ready recipe for building live, push-based APIs — the same
building blocks used throughout this whole course, just put together for
the SSE use case specifically.
