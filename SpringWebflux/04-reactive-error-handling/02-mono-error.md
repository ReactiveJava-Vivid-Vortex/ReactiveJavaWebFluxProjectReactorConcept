# Mono.error()

## In Simple Terms

`Mono.error(exception)` is how you signal a failure from inside a reactive
service method — the reactive version of `throw`, just expressed as a
returned value instead of an actual thrown exception, so it fits naturally
into the rest of the chain.

## Simple Example

```java
public Mono<ProductDto> getProduct(String id) {
    if (id == null || id.isBlank()) {
        return Mono.error(new IllegalArgumentException("Product id must not be blank"));
    }

    return productRepository.findById(id)
        .switchIfEmpty(Mono.error(new ProductNotFoundException(id)))
        .map(ProductMapper::toDto);
}
```

The error travels all the way to whoever eventually subscribes — in a
WebFlux app, that's the framework itself, which routes it to your
`@ControllerAdvice` (or the default error handling) to build the right
HTTP response.

## Why It Matters

Using `Mono.error()` (rather than literally `throw`-ing inside a lambda,
which can behave in surprising ways depending on where it happens in a
reactive chain) is the reliable, idiomatic way to signal a failure from
inside a reactive method, making sure it's properly captured as part of the
`Mono`'s error channel.
