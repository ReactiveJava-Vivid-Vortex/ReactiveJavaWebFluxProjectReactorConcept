# Mono (in Spring WebFlux)

## In Simple Terms

In a Spring WebFlux controller, `Mono<T>` is the return type you use whenever an
endpoint produces **at most one** result — a single entity, a boolean success flag,
or nothing at all (e.g., `Mono<Void>` for a DELETE operation). Spring WebFlux
subscribes to the `Mono` you return and streams the resulting single value back to
the HTTP client as the response body once it's available.

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

If `repository.findById(id)` completes empty (product not found), Spring WebFlux
automatically responds with `404 Not Found` by default for a `Mono` that resolves
empty in a controller method.

## Why It Matters

Using `Mono` correctly in controller signatures is the foundation of writing
non-blocking WebFlux endpoints — it tells Spring (and future readers of your code)
exactly how many results to expect, and lets the framework handle empty/error cases
according to well-defined, consistent conventions.
