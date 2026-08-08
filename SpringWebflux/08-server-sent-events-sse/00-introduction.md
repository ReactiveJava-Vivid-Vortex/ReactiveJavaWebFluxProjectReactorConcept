# Q1. What Is SSE, and Why Not Just Poll Every Second?

## Simple Explanation (Think of a News Alert Push vs Refreshing a Webpage)

Polling is like refreshing a news website every second, hoping something new
appeared — wasteful, most refreshes find nothing new. **SSE** is a **push
notification**: the server tells you the instant something actually happens, over
one connection that just stays open.

```
Polling:  GET /price -> nothing changed. GET /price -> nothing changed.
          GET /price -> nothing changed. GET /price -> CHANGED! (wasted 3 calls)

SSE:      ONE connection stays open. Server pushes ONLY when the price actually changes.
```

```java
@GetMapping(value = "/stock-price/{symbol}", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<StockPrice> streamPrice(@PathVariable String symbol) {
    return priceService.getLivePriceUpdates(symbol);
}
```

```javascript
const eventSource = new EventSource('/stock-price/AAPL');
eventSource.onmessage = (event) => console.log('Price update:', event.data);
// Browser auto-reconnects if the connection drops — zero extra code needed!
```

---

## Q2. Why "Server-Sent," Specifically? (Direction Matters)

SSE is **one-directional**: server → client only. If you need the client to also
send messages back over the same live connection, you need WebSockets instead —
SSE is the simpler tool when you only need server-to-browser push.

---

## Q3. Is an SSE Endpoint Finite or Infinite?

Almost always **infinite** — it typically never calls `onComplete()` on its own,
just like `Flux.interval()`. It keeps producing events as long as the connection
stays open.

```java
@GetMapping(value = "/notifications", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<String> streamNotifications() {
    return Flux.interval(Duration.ofSeconds(3)).map(tick -> "Notification #" + tick);
}
```

When the browser tab closes, WebFlux **automatically cancels** the underlying
`Flux` — see [[cancellation-of-requests]] in the Reactive Programming Fundamentals
topic — so nothing keeps running server-side for an absent client.

---

## Q4. How Do I Power SSE with REAL Application Events (Not Just a Timer)?

```java
@Service
public class NotificationService {
    private final Sinks.Many<String> sink = Sinks.many().multicast().onBackpressureBuffer();

    public void notify(String message) { sink.tryEmitNext(message); }

    public Flux<ServerSentEvent<String>> subscribe() {
        return sink.asFlux().map(msg -> ServerSentEvent.<String>builder().data(msg).build());
    }
}
```

Every connected browser calls `subscribe()`, and any code anywhere can call
`.notify(...)` to broadcast a live event to all of them — see the Sinks topic in
the Project Reactor notes for the full mechanics of `Sinks.Many`.

---

## Q5. Interview-Style Q&A

### Does the browser need a special library to consume SSE?

**No** — `EventSource` is a built-in browser API, with automatic reconnection
handled for you.

### Can SSE send data from the client back to the server?

**No** — it's one-directional (server → client). For bidirectional communication,
use WebSockets instead.

### Is an SSE `Flux` typically hot or cold?

Typically **hot**, backed by a `Sinks.Many` — new subscribers only see events from
the moment they connect onward, unless replay is explicitly configured (see the
Hot & Cold Publishers topic).

---

## Q6. Summary

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

### One sentence to remember

> **"SSE is a push notification over plain HTTP — one open connection, server
> pushes only when something actually happens, browser consumes it with zero
> extra libraries."**
