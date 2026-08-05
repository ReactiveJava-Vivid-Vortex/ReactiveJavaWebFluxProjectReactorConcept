# Flux (in Spring WebFlux)

## In Simple Terms

In a Spring WebFlux controller, `Flux<T>` is the return type for endpoints that
produce **zero or many** results — a list of entities, streamed one at a time. By
default, Spring serializes a `Flux<T>` response as a JSON array, but it can also be
configured to stream results incrementally (e.g., as newline-delimited JSON, or
Server-Sent Events).

## Simple Example

```java
@RestController
@RequestMapping("/products")
public class ProductController {

    private final ProductRepository repository;

    @GetMapping
    public Flux<Product> getAllProducts() {
        return repository.findAll(); // Flux<Product> — 0 to N results
    }

    @GetMapping(value = "/stream", produces = MediaType.APPLICATION_NDJSON_VALUE)
    public Flux<Product> streamProducts() {
        return repository.findAll(); // same source, streamed as NDJSON instead of one JSON array
    }
}
```

With the default `produces = APPLICATION_JSON_VALUE`, Spring WebFlux waits for the
whole `Flux` to complete before writing the complete JSON array response. With
`APPLICATION_NDJSON_VALUE`, each item is written to the response as soon as it's
available.

## Why It Matters

Choosing `Flux` correctly (and its produced media type) directly affects your API's
behavior under large datasets — a plain JSON array response still needs the full
`Flux` to complete before anything is sent, while a streaming media type lets clients
start processing data as it arrives, which matters a lot for large or slow-to-produce
result sets.
