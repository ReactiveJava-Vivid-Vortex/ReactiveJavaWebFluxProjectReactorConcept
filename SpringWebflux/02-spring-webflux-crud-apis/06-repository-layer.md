# Repository Layer

## In Simple Terms

The repository layer is responsible for **reactive data access** — talking to the
database and returning `Mono`/`Flux` results. In Spring Data R2DBC, you define
repository interfaces extending `ReactiveCrudRepository`, and Spring generates the
implementation for you, just like in traditional Spring Data JPA, but fully
non-blocking.

## Simple Example

```java
public interface ProductRepository extends ReactiveCrudRepository<ProductEntity, String> {

    // Custom query method - Spring Data generates the implementation
    Flux<ProductEntity> findByNameContaining(String keyword);

    // Custom query with @Query annotation
    @Query("SELECT * FROM products WHERE cost BETWEEN :min AND :max")
    Flux<ProductEntity> findByPriceRange(double min, double max);
}
```

Usage in a service:

```java
public Flux<ProductEntity> searchProducts(String keyword) {
    return productRepository.findByNameContaining(keyword);
}
```

Built-in methods from `ReactiveCrudRepository` include `findById()`, `findAll()`,
`save()`, `deleteById()`, `count()`, and more — all returning `Mono`/`Flux`.

## Why It Matters

The repository layer is where the non-blocking, reactive nature of your data access
begins — every method here returns a `Mono`/`Flux`, ensuring the rest of your
application (service and controller layers) can compose on top of it reactively,
without ever needing to block waiting for database results.
