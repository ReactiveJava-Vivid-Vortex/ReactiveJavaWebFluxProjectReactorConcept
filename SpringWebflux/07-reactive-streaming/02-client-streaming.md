# Client Streaming

## In Simple Terms

"Client streaming" means your WebFlux endpoint accepts a **request body that itself
streams data incrementally** — e.g., a client uploading a large number of records
one at a time — rather than requiring the entire request body to be fully received
before processing begins.

## Simple Example

```java
@PostMapping(value = "/products/upload", consumes = MediaType.APPLICATION_NDJSON_VALUE)
public Mono<UploadSummary> uploadProducts(@RequestBody Flux<ProductDto> productStream) {
    return productStream
        .map(ProductMapper::toEntity)
        .flatMap(productRepository::save)
        .count()
        .map(count -> new UploadSummary(count, "Upload successful"));
}
```

As the client sends each NDJSON line, the server processes and saves it
incrementally — it doesn't need to buffer the entire request body in memory before
starting work, even if the client is uploading millions of records.

## Why It Matters

Client streaming is essential for handling large uploads efficiently — without it,
processing a huge upload would require buffering the entire request body in memory
first, which doesn't scale for very large or continuous data feeds (e.g., a client
continuously streaming sensor readings to the server).
