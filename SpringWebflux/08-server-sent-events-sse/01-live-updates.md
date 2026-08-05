# Live Updates

## In Simple Terms

"Live updates" refers to pushing new information to a connected client as soon as
it happens, without the client needing to repeatedly ask ("poll") for it. Server-Sent
Events (SSE) is one of the simplest ways to implement live updates in a WebFlux
application — the server keeps a connection open and pushes new data whenever it's
available.

## Simple Example

```java
@GetMapping(value = "/stock-price/{symbol}", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<StockPrice> streamPrice(@PathVariable String symbol) {
    return priceService.getLivePriceUpdates(symbol); // Flux<StockPrice> - pushes on every change
}
```

A browser connecting to this endpoint receives a new event automatically every time
the price changes — no repeated polling requests (`GET /stock-price/AAPL` every
second) needed.

## Why It Matters

Live updates via SSE are far more efficient than client-side polling: polling wastes
resources sending repeated requests even when nothing has changed, whereas SSE only
sends data when there's actually something new to report — while still using plain
HTTP, no special infrastructure (like WebSockets) required.
