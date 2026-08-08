# Flux (in Spring WebFlux)

## In Simple Terms

In a WebFlux controller, `Flux<T>` is what you return when an endpoint can
produce zero, one, or many results — a list of records, streamed out one
at a time. By default, Spring turns a `Flux<T>` response into a JSON
array, but you can also set it up to stream results incrementally instead
(as newline-delimited JSON, or Server-Sent Events).

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

With the default `produces = APPLICATION_JSON_VALUE`, WebFlux waits for
the whole `Flux` to finish before writing out the complete JSON array. With
`APPLICATION_NDJSON_VALUE`, each item gets written to the response the
moment it's ready.

## Why It Matters

Picking `Flux` correctly (and choosing the right response media type)
directly changes how your API behaves on large datasets — a plain JSON
array still needs the whole `Flux` to finish before anything gets sent,
while a streaming format lets clients start working with data as it
arrives, which matters a lot for big or slow-to-produce results.
