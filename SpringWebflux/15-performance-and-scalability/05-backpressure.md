# Backpressure (WebFlux Performance)

## In Simple Terms

In the context of WebFlux performance, backpressure ensures that streaming
endpoints (large downloads, NDJSON exports) never overwhelm slow clients or
overload server memory — the underlying Reactive Streams machinery (Netty +
Project Reactor) automatically paces data production to match what the client can
actually consume.

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

You rarely need to configure backpressure explicitly for typical WebFlux endpoints
— it's handled transparently by the underlying Reactive Streams implementation, as
long as your entire pipeline (repository, mapping, response writing) stays reactive
and doesn't introduce an unbounded buffering point (like an accidental
`.collectList()` on a huge stream).

## Why It Matters

Backpressure is what allows WebFlux to safely stream very large or slow-to-produce
datasets to clients with varying network speeds, without risking server memory
exhaustion — a performance and reliability guarantee that a naive
"buffer everything, then send" approach could never provide at scale.
