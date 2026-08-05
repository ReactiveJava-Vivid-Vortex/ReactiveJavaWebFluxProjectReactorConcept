# Server Sent Events (SSE) — Topic Overview

## What Is This Topic About? (In Simple Terms)

SSE is the simplest way to push **live updates** from a WebFlux server to a browser
— no WebSockets, no extra libraries, just a regular HTTP response that stays open
and streams events over time. You return a `Flux<T>` with the special
`text/event-stream` media type, and the browser's built-in `EventSource` API
consumes it directly:

```java
@GetMapping(value = "/stock-price/{symbol}", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<StockPrice> streamPrice(@PathVariable String symbol) {
    return priceService.getLivePriceUpdates(symbol); // pushes on every price change
}
```

```javascript
const eventSource = new EventSource('/stock-price/AAPL');
eventSource.onmessage = (event) => console.log('Price update:', event.data);
// automatically reconnects on connection drop — no extra code needed!
```

This is dramatically more efficient than the alternative — client-side polling
(repeatedly calling `GET /stock-price/AAPL` every second, wasting requests even when
nothing changed). SSE endpoints are naturally **continuous data streams** — they
often never complete on their own (like `Flux.interval()`) — and WebFlux
automatically cancels the underlying pipeline the moment the browser tab closes, so
nothing is wasted on an absent client.

For a real application (not just a timer demo), you back the stream with a
`Sinks.Many` — application code calls `sink.tryEmitNext(event)` whenever something
actually happens, and every connected browser receives it live.

## Quick Revision Cheat Sheet

| # | Concept | One-Line Summary |
|---|---|---|
| 1 | **Live updates** | Push new data to a client as it happens — no repeated polling requests needed. |
| 2 | **Continuous data stream** | An SSE endpoint that keeps the connection open, emitting events over time (often never completes on its own). |
| 3 | **Browser EventSource** | Built-in browser JS API for consuming `text/event-stream` — auto-parses events and auto-reconnects. |
| 4 | **Reactive streaming APIs** | Combine `Sinks.Many` (to broadcast real events) + `ServerSentEvent<T>` (for named, resumable events) + the SSE media type. |

## How It All Fits Together

```
Application event happens (price change, new message, etc.)
        │
        ▼
sink.tryEmitNext(event)     ← Sinks.Many broadcasting to all listeners
        │
        ▼
@GetMapping(produces = TEXT_EVENT_STREAM_VALUE)
public Flux<T> streamEvents() { return sink.asFlux(); }
        │
        ▼
Browser: new EventSource('/endpoint') → eventSource.onmessage = ...
        (auto-reconnects if the connection drops)
```

Reach for SSE specifically when the consumer is a **browser** and communication is
**one-directional** (server → client) — for bidirectional communication, or non-browser
clients, plain NDJSON streaming or WebSockets may be a better fit.
