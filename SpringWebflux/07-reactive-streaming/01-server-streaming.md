# Server Streaming

## In Simple Terms

"Server streaming" means a WebFlux endpoint sends its response **incrementally**, as
data becomes available, rather than waiting for the entire result set to be ready
before sending anything. This is achieved by returning a `Flux<T>` with a streaming
media type (like NDJSON or SSE) instead of the default JSON array format.

## Simple Example

```java
@GetMapping(value = "/products/stream", produces = MediaType.APPLICATION_NDJSON_VALUE)
public Flux<ProductDto> streamProducts() {
    return productRepository.findAll() // a Flux<ProductEntity>, potentially huge
        .map(ProductMapper::toDto);
}
```

With `APPLICATION_NDJSON_VALUE`, each `ProductDto` is written to the HTTP response as
a separate line of JSON, as soon as it's available from the database — the client can
start processing results immediately, rather than waiting for the entire dataset.

Compare with the default (non-streaming) JSON array behavior:

```java
@GetMapping("/products") // default produces = APPLICATION_JSON_VALUE
public Flux<ProductDto> getAllProducts() {
    return productRepository.findAll().map(ProductMapper::toDto);
    // Waits for the ENTIRE Flux to complete before writing the full JSON array
}
```

## Why It Matters

Server streaming is essential for large datasets, slow-to-produce results, or
real-time feeds — it reduces time-to-first-byte dramatically and lets clients begin
processing data as it arrives, rather than waiting for a potentially very large,
fully-buffered response.
