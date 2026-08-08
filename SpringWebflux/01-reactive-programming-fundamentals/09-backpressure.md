# Backpressure

## In Simple Terms

Backpressure lets a slow consumer (say, a client on a slow network
connection) control how fast a fast producer (say, a server streaming a
huge file) sends it data — so the consumer never gets flooded with more
than it can actually handle.

## Simple Example

In a Spring WebFlux streaming endpoint:

```java
@GetMapping(value = "/large-dataset", produces = MediaType.APPLICATION_NDJSON_VALUE)
public Flux<Record> streamRecords() {
    return recordRepository.findAll(); // Flux<Record>, potentially millions of rows
}
```

If the HTTP client (or its network) can only take data slowly, the
underlying Netty/Reactor machinery naturally applies backpressure — the
database query only fetches as much as the client can currently handle,
instead of pulling everything into server memory upfront and hoping the
client can keep up.

## Why It Matters

Without backpressure, a fast server could stream data much faster than a
slow client (or network) can receive it, forcing the server to pile up
huge amounts of unsent data in memory. Backpressure — baked into the
Reactive Streams spec that WebFlux is built on — keeps memory usage
bounded and tied to actual throughput, even when producer and consumer
speeds are wildly different.
