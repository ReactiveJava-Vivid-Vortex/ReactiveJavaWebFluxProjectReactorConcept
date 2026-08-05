# switchIfEmpty()

## In Simple Terms

`.switchIfEmpty(fallback)` is the standard way to convert an "empty" result (like a
`findById()` that finds nothing) into an explicit error, or into an alternate
`Mono` — a very common pattern for implementing "not found" behavior in reactive
service methods.

## Simple Example

```java
public Mono<ProductDto> getProduct(String id) {
    return productRepository.findById(id)
        .switchIfEmpty(Mono.error(new ProductNotFoundException(id))) // empty -> error
        .map(ProductMapper::toDto);
}
```

Alternatively, falling back to a default value instead of an error:

```java
public Mono<ProductDto> getProductOrDefault(String id) {
    return productRepository.findById(id)
        .map(ProductMapper::toDto)
        .switchIfEmpty(Mono.just(ProductDto.placeholder())); // empty -> fallback value
}
```

## Why It Matters

`.switchIfEmpty()` is the idiomatic way to turn "nothing found" into a well-defined
outcome — either a specific, meaningful error (handled centrally via
`@ControllerAdvice`) or a sensible default — rather than letting an empty `Mono`
silently produce an ambiguous, empty HTTP response.
