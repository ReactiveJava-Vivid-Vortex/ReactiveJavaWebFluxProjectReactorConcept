# Streaming Millions of Records

## In Simple Terms

Combine reactive repositories, NDJSON, and backpressure, and a WebFlux
endpoint can stream absolutely huge datasets (millions of rows) to a
client while keeping server memory roughly steady the whole time — instead
of loading the entire result set into a `List` first.

## Simple Example

```java
@GetMapping(value = "/products/export", produces = MediaType.APPLICATION_NDJSON_VALUE)
public Flux<ProductDto> exportAllProducts() {
    return productRepository.findAll() // Flux<ProductEntity> - streamed from DB, not pre-loaded
        .map(ProductMapper::toDto)
        .doOnNext(dto -> log.debug("Streaming product: {}", dto.id()));
}
```

Because the R2DBC repository itself hands back a `Flux` (not a
pre-loaded `List`), rows get pulled from the database and streamed to the
client incrementally — the server never holds all 1,000,000+ rows in
memory at once.

The same idea applies in reverse for uploading millions of records, using
a `Flux<ProductDto>` request body (see [[client-streaming]]).

## Why It Matters

This combo — reactive repository plus a streaming media type — is what
makes it realistic to export or import huge datasets over a plain HTTP
API, without needing special batch-transfer infrastructure or risking an
`OutOfMemoryError` on either end.
