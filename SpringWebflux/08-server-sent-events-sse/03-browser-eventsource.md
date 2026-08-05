# Browser EventSource

## In Simple Terms

`EventSource` is a built-in browser JavaScript API specifically designed to consume
`text/event-stream` (SSE) endpoints — no external libraries needed. It automatically
handles the connection, parses incoming events, and even automatically reconnects if
the connection drops.

## Simple Example

Server (Spring WebFlux):

```java
@GetMapping(value = "/notifications", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<String> streamNotifications() {
    return Flux.interval(Duration.ofSeconds(3))
        .map(tick -> "Notification #" + tick);
}
```

Client (plain browser JavaScript, no libraries):

```javascript
const eventSource = new EventSource('/notifications');

eventSource.onmessage = (event) => {
    console.log('Received:', event.data);
};

eventSource.onerror = (error) => {
    console.error('Connection error:', error);
    // EventSource automatically attempts to reconnect
};
```

## Why It Matters

`EventSource`'s built-in reconnection logic and simple API make SSE an especially
low-friction choice for browser-facing live updates — unlike WebSockets, which
require a dedicated client library/protocol handling, SSE works with a native
browser API in just a few lines of JavaScript.
