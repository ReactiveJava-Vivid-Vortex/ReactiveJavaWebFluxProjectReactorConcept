# Custom Exceptions

## In Simple Terms

Custom exceptions are your own exception classes representing specific,
meaningful failure conditions in your domain (e.g., `ProductNotFoundException`,
`InsufficientStockException`) — rather than relying on generic exceptions like
`RuntimeException`. They make error handling in your reactive pipelines (and in
`@ControllerAdvice`) far more precise and readable.

## Simple Example

```java
public class ProductNotFoundException extends RuntimeException {
    private final String productId;

    public ProductNotFoundException(String productId) {
        super("Product not found: " + productId);
        this.productId = productId;
    }

    public String getProductId() {
        return productId;
    }
}

public class InsufficientStockException extends RuntimeException {
    public InsufficientStockException(String productId, int requested, int available) {
        super(String.format("Insufficient stock for %s: requested %d, available %d",
            productId, requested, available));
    }
}
```

Using them inside a reactive pipeline:

```java
public Mono<ProductDto> getProduct(String id) {
    return productRepository.findById(id)
        .switchIfEmpty(Mono.error(new ProductNotFoundException(id)))
        .map(ProductMapper::toDto);
}
```

## Why It Matters

Custom exceptions give your error-handling code (both `onErrorResume()` calls and
centralized `@ControllerAdvice` handlers) something specific and meaningful to match
against — instead of parsing generic exception messages, you can branch on exact
exception types, each mapped to the correct HTTP status and error response.
