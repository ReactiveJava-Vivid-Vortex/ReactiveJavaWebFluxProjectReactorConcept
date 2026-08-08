# Mono (in Spring WebFlux)

## In Simple Terms

In a WebFlux controller, `Mono<T>` is what you return whenever an endpoint
gives back at most one result — a single record, a true/false flag, or
nothing at all (`Mono<Void>` for something like a DELETE). WebFlux
subscribes to whatever `Mono` you return and streams that one value back to
the client as the response once it's ready.

## Simple Example

```java
@RestController
@RequestMapping("/products")
public class ProductController {

    private final ProductRepository repository;

    @GetMapping("/{id}")
    public Mono<Product> getProduct(@PathVariable String id) {
        return repository.findById(id); // Mono<Product> — 0 or 1 result
    }

    @DeleteMapping("/{id}")
    public Mono<Void> deleteProduct(@PathVariable String id) {
        return repository.deleteById(id); // Mono<Void> — no meaningful result
    }
}
```

If `repository.findById(id)` comes back empty (product not found), Spring
WebFlux automatically responds with `404 Not Found` by default for an
empty `Mono` returned from a controller method.

## Why It Matters

Using `Mono` correctly in your controller signatures is the foundation for
writing non-blocking endpoints — it tells Spring (and anyone reading your
code later) exactly how many results to expect, and lets the framework
handle empty/error cases in a predictable, consistent way.
