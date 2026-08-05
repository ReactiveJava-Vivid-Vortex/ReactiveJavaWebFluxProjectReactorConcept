# Streaming Millions of Records

## In Simple Terms

Combining reactive repositories, NDJSON, and backpressure allows a WebFlux endpoint
to stream extremely large datasets (millions of rows) to a client while keeping
server memory usage roughly constant throughout — as opposed to loading the entire
result set into a `List` first.

## Simple Example

```java
@GetMapping(value = "/products/export", produces = MediaType.APPLICATION_NDJSON_VALUE)
public Flux<ProductDto> exportAllProducts() {
    return productRepository.findAll() // Flux<ProductEntity> - streamed from DB, not pre-loaded
        .map(ProductMapper::toDto)
        .doOnNext(dto -> log.debug("Streaming product: {}", dto.id()));
}
```

Because the R2DBC repository itself returns a `Flux` (not a pre-materialized
`List`), rows are fetched from the database and streamed to the client incrementally
— the server never holds all 1,000,000+ rows in memory simultaneously.

For the reverse direction (uploading millions of records), the same principle
applies using a `Flux<ProductDto>` request body (see [[client-streaming]]).

## Why It Matters

This pattern — reactive repository + streaming media type — is what makes it
practical to export/import extremely large datasets through a standard HTTP API,
without needing specialized batch-file-transfer infrastructure or risking
`OutOfMemoryError` on either the server or client.
