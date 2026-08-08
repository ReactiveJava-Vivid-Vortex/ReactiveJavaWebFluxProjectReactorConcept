# Browser EventSource

## In Simple Terms

`EventSource` is a browser JavaScript API built specifically for consuming
`text/event-stream` (SSE) endpoints — no extra libraries needed. It
handles the connection for you, parses incoming events, and even
automatically reconnects if the connection drops.

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

`EventSource`'s built-in reconnection and simple API make SSE a
low-friction choice for pushing live updates to a browser — unlike
WebSockets, which need their own client library and protocol handling, SSE
works with a browser API that's already there, in just a few lines of
JavaScript.
