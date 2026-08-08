# Continuous Data Stream

## In Simple Terms

A "continuous data stream" is an SSE endpoint that keeps its connection
open for a long time (or forever), constantly sending events — unlike a
normal HTTP response, which finishes once it's sent. In WebFlux, this
naturally comes out as a `Flux` that doesn't necessarily ever complete.

## Simple Example

```java
@GetMapping(value = "/system-metrics", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<SystemMetrics> streamMetrics() {
    return Flux.interval(Duration.ofSeconds(2))
        .map(tick -> SystemMetrics.captureSnapshot()); // never completes on its own
}
```

The client gets a fresh `SystemMetrics` snapshot every 2 seconds,
indefinitely, until it disconnects (at which point WebFlux automatically
shuts down the underlying `Flux.interval()` — see
[[cancellation-of-requests]]).

## Why It Matters

Continuous data streams power dashboards, monitoring displays, and any UI
that needs to reflect changing server-side state close to real time — the
reactive model handles this naturally, since an SSE endpoint returning
`Flux<T>` is exactly the same programming model as any other WebFlux
endpoint, just with a media type that keeps the connection open.
