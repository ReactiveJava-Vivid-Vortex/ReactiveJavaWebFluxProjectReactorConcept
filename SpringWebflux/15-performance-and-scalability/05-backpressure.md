# Backpressure (WebFlux Performance)

## In Simple Terms

For WebFlux performance, backpressure makes sure streaming endpoints
(large downloads, NDJSON exports) never overwhelm a slow client or flood
server memory — the underlying Reactive Streams machinery (Netty + Project
Reactor) automatically paces how fast data gets produced to match what the
client can actually handle.

## Simple Example

```java
@GetMapping(value = "/products/export", produces = MediaType.APPLICATION_NDJSON_VALUE)
public Flux<ProductDto> exportProducts() {
    return productRepository.findAll() // potentially millions of rows
        .map(ProductMapper::toDto);
    // Backpressure automatically applies: if the client (or its network connection)
    // is slow, the database query itself is paced to match, rather than the
    // server buffering unsent data in memory
}
```

You rarely have to configure backpressure yourself for typical WebFlux
endpoints — it's handled transparently by the underlying Reactive Streams
setup, as long as your whole pipeline (repository, mapping, response
writing) stays reactive and doesn't sneak in an unbounded buffering point
(like an accidental `.collectList()` on a huge stream).

## Why It Matters

Backpressure is what lets WebFlux safely stream very large or
slow-to-produce datasets to clients with all kinds of network speeds,
without risking server memory running out — a guarantee a naive "buffer
everything, then send" approach could never give you at scale.
