# Live Updates

## In Simple Terms

"Live updates" means pushing new information to a connected client the
moment it happens, instead of the client having to keep asking ("polling")
for it. Server-Sent Events (SSE) is one of the easiest ways to do live
updates in WebFlux — the server just keeps the connection open and pushes
new data whenever there's something to send.

## Simple Example

```java
@GetMapping(value = "/stock-price/{symbol}", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<StockPrice> streamPrice(@PathVariable String symbol) {
    return priceService.getLivePriceUpdates(symbol); // Flux<StockPrice> - pushes on every change
}
```

A browser connected to this endpoint automatically gets a new event every
time the price changes — no repeated `GET /stock-price/AAPL` requests every
second needed.

## Why It Matters

Live updates through SSE are much more efficient than the client polling
over and over: polling wastes resources sending requests even when nothing
changed, while SSE only sends data when there's actually something new —
and it does all this over plain HTTP, no special infrastructure (like
WebSockets) required.
