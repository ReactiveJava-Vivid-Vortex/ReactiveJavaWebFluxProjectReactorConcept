# Custom Exceptions

## In Simple Terms

Custom exceptions are your own exception classes for specific, meaningful
failures in your app — `ProductNotFoundException`,
`InsufficientStockException` — instead of leaning on generic ones like
`RuntimeException`. They make error handling (both `onErrorResume()` calls
and centralized `@ControllerAdvice` handlers) much clearer and more
precise.

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

Custom exceptions give your error-handling code something specific and
meaningful to check against — instead of parsing generic error messages,
you can branch on exact exception types, each mapped to the right HTTP
status and response.
