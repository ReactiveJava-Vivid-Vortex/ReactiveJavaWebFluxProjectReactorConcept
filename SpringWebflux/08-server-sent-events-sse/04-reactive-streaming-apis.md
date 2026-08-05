# Reactive Streaming APIs

## In Simple Terms

Building a well-designed SSE-based API in WebFlux typically involves combining
several pieces covered elsewhere: a `Sinks.Many` to broadcast events, a `Flux`
returned from a controller with `produces = TEXT_EVENT_STREAM_VALUE`, and often
named events using `ServerSentEvent<T>` for more structured client-side handling.

## Simple Example

Using `ServerSentEvent` for named events with IDs (supporting automatic
reconnection resumption):

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

Backing it with a `Sinks.Many` for broadcasting real application events (not just a
timer):

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

Combining `Sinks` (for event production), `ServerSentEvent` (for structured,
resumable events), and streaming media types gives you a complete, production-ready
pattern for building live, push-based APIs — the same building blocks used across
this entire course, composed together for the specific SSE use case.
