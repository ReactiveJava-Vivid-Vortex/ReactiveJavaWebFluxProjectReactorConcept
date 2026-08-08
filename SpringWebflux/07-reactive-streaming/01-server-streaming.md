# Server Streaming

## In Simple Terms

"Server streaming" means a WebFlux endpoint sends its response bit by
bit, as data becomes ready, instead of waiting for the whole result set
before sending anything. You do this by returning a `Flux<T>` with a
streaming media type (like NDJSON or SSE) instead of the default JSON
array.

## Simple Example

```java
@GetMapping(value = "/products/stream", produces = MediaType.APPLICATION_NDJSON_VALUE)
public Flux<ProductDto> streamProducts() {
    return productRepository.findAll() // a Flux<ProductEntity>, potentially huge
        .map(ProductMapper::toDto);
}
```

With `APPLICATION_NDJSON_VALUE`, each `ProductDto` gets written to the
response as its own line of JSON, the moment it's ready from the database
— the client can start working with results right away, instead of
waiting for everything.

Compare with the default (non-streaming) JSON array behavior:

```java
@GetMapping("/products") // default produces = APPLICATION_JSON_VALUE
public Flux<ProductDto> getAllProducts() {
    return productRepository.findAll().map(ProductMapper::toDto);
    // Waits for the ENTIRE Flux to complete before writing the full JSON array
}
```

## Why It Matters

Server streaming matters a lot for big datasets, slow-to-produce results,
or real-time feeds — it dramatically cuts down time-to-first-byte and lets
clients start processing data as it arrives, instead of waiting on a
potentially huge, fully-buffered response.
