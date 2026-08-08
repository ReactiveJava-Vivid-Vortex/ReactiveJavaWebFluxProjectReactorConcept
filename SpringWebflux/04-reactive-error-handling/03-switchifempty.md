# switchIfEmpty()

## In Simple Terms

`.switchIfEmpty()` is the standard way to turn "nothing came back" (like a
`findById()` that found nothing) into a proper error, or into some
alternate result — a very common pattern for implementing "not found"
behavior.

## Simple Example

```java
public Mono<ProductDto> getProduct(String id) {
    return productRepository.findById(id)
        .switchIfEmpty(Mono.error(new ProductNotFoundException(id))) // empty -> error
        .map(ProductMapper::toDto);
}
```

Or falling back to a default value instead of an error:

```java
public Mono<ProductDto> getProductOrDefault(String id) {
    return productRepository.findById(id)
        .map(ProductMapper::toDto)
        .switchIfEmpty(Mono.just(ProductDto.placeholder())); // empty -> fallback value
}
```

## Why It Matters

`.switchIfEmpty()` is the natural way to turn "nothing found" into a
clearly defined outcome — either a meaningful error (handled centrally via
`@ControllerAdvice`) or a sensible default — instead of letting an empty
`Mono` quietly produce a vague, empty response.
