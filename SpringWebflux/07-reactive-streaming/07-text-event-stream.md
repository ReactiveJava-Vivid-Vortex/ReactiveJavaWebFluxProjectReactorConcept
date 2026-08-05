# text/event-stream

## In Simple Terms

`text/event-stream` is the standard MIME type for **Server-Sent Events (SSE)** — a
format designed specifically for a server continuously pushing text-based events to
a browser client over a single, long-lived HTTP connection. It's distinct from
NDJSON: SSE has its own simple wire format (`data: ...\n\n`) and built-in browser
support via the `EventSource` API.

## Simple Example

```java
@GetMapping(value = "/notifications", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<String> streamNotifications() {
    return Flux.interval(Duration.ofSeconds(5))
        .map(tick -> "New notification at " + Instant.now());
}
```

The raw bytes sent to the client look like:

```
data: New notification at 2026-08-05T10:00:00Z

data: New notification at 2026-08-05T10:00:05Z

```

A browser can consume this directly with the built-in `EventSource` JavaScript API,
without any special libraries.

## Why It Matters

`text/event-stream` is specifically built for **server-to-client push** scenarios
(live notifications, price updates, progress bars) where a browser needs to receive
ongoing updates over time — it's simpler to consume from a browser than raw NDJSON
(no special client library needed) and is covered in depth in the dedicated
[[live-updates]] section on Server-Sent Events.
