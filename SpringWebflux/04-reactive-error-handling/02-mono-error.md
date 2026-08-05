# Mono.error()

## In Simple Terms

`Mono.error(exception)` is how you signal a failure from within a reactive service
method — the reactive equivalent of `throw`, but expressed as a returned value
instead of an actual thrown exception, so it composes naturally with the rest of the
chain.

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

The error propagates all the way to whoever eventually subscribes — in a WebFlux
app, that's the framework itself, which routes it to your `@ControllerAdvice` (or
default error handling) to produce the appropriate HTTP response.

## Why It Matters

Using `Mono.error()` (rather than literally `throw`-ing inside a lambda, which can
sometimes behave unexpectedly in reactive contexts depending on where it happens) is
the idiomatic, reliable way to signal a failure from within a reactive method,
ensuring it's properly captured as part of the `Mono`'s error channel.
