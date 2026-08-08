# Client Streaming

## In Simple Terms

"Client streaming" means your endpoint accepts a request body that itself
arrives incrementally — a client uploading lots of records one at a time —
instead of requiring the whole body to show up before processing starts.

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
right away — it doesn't need to hold the whole request body in memory
first, even if the client is uploading millions of records.

## Why It Matters

Client streaming is what makes handling big uploads practical — without
it, processing a huge upload would mean buffering the entire body in
memory first, which just doesn't scale for very large or ongoing data
feeds (like a client continuously streaming sensor readings to the
server).
