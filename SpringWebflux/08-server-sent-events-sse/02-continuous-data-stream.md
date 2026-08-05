# Continuous Data Stream

## In Simple Terms

A "continuous data stream" is an SSE endpoint that keeps the connection open
indefinitely (or for a long time), continuously emitting events over time — as
opposed to a normal HTTP response that completes once. In WebFlux, this is naturally
expressed as a `Flux` that doesn't necessarily ever complete.

## Simple Example

```java
@GetMapping(value = "/system-metrics", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<SystemMetrics> streamMetrics() {
    return Flux.interval(Duration.ofSeconds(2))
        .map(tick -> SystemMetrics.captureSnapshot()); // never completes on its own
}
```

The client receives a fresh `SystemMetrics` snapshot every 2 seconds, indefinitely,
until it disconnects (at which point WebFlux automatically cancels the underlying
`Flux.interval()` — see [[cancellation-of-requests]]).

## Why It Matters

Continuous data streams power dashboards, monitoring displays, and any UI that
needs to reflect changing server-side state in near real time — the reactive model
handles this naturally, since an SSE endpoint returning `Flux<T>` is exactly the same
programming model as any other WebFlux endpoint, just with a media type that keeps
the connection open.
