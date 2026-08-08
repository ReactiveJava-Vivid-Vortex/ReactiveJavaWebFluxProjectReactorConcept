# text/event-stream

## In Simple Terms

`text/event-stream` is the standard content type for Server-Sent Events
(SSE) — a format built specifically for a server continuously pushing
text-based updates to a browser over one long-lived connection. It's
different from NDJSON: SSE has its own simple wire format (`data: ...\n\n`)
and browsers already know how to consume it natively through the
`EventSource` API.

## Simple Example

```java
@GetMapping(value = "/notifications", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<String> streamNotifications() {
    return Flux.interval(Duration.ofSeconds(5))
        .map(tick -> "New notification at " + Instant.now());
}
```

The raw bytes sent to the client look like this:

```
data: New notification at 2026-08-05T10:00:00Z

data: New notification at 2026-08-05T10:00:05Z

```

A browser can read this straight away with the built-in `EventSource`
JavaScript API — no special library needed.

## Why It Matters

`text/event-stream` is built exactly for server-to-client push scenarios
(live notifications, price updates, progress bars) where a browser needs
ongoing updates over time — it's simpler to consume from a browser than
raw NDJSON, and gets covered in more depth in the dedicated
[[live-updates]] section on Server-Sent Events.
