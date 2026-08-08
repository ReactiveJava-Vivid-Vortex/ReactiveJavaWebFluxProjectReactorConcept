# Exception Factory

## In Simple Terms

An "exception factory" is a small helper — a static method or a dedicated
class — that centralizes how your custom exceptions get built, so the
messages and construction stay consistent everywhere, instead of copying
slightly different `new SomeException(...)` calls throughout your code.

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

An exception factory keeps you from ending up with mismatched error
messages scattered around ("Product not found" vs "product doesn't exist"
vs "No such product"), and makes it easy to update the wording or logic
for a given error in one place, instead of hunting down every
`new SomeException(...)` call.
