# Exception Factory

## In Simple Terms

An "exception factory" is a small helper (a static method, or a dedicated class)
that centralizes the creation of your custom exceptions — ensuring consistent
messages and construction logic across your codebase, rather than duplicating
`new SomeException(...)` calls with slightly different message formats everywhere.

## Simple Example

```java
public final class ProductExceptions {

    private ProductExceptions() {}

    public static ProductNotFoundException notFound(String id) {
        return new ProductNotFoundException("Product not found with id: " + id);
    }

    public static InvalidProductException invalidPrice(double price) {
        return new InvalidProductException(
            "Invalid price: " + price + " (must be positive)"
        );
    }
}
```

Usage — cleaner, consistent, and centralized:

```java
public Mono<ProductDto> getProduct(String id) {
    return productRepository.findById(id)
        .switchIfEmpty(Mono.error(ProductExceptions.notFound(id)))
        .map(ProductMapper::toDto);
}
```

## Why It Matters

An exception factory prevents inconsistent error messages scattered across a
codebase (e.g., "Product not found" vs "product doesn't exist" vs "No such
product") and makes it easy to update messaging/logic for a given error type in one
place, rather than hunting down every `new SomeException(...)` call site.
