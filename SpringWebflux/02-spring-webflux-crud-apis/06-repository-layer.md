# Repository Layer

## In Simple Terms

The repository layer talks to the database and hands back `Mono`/`Flux`
results — this is where reactive data access happens. With Spring Data
R2DBC, you write repository interfaces that extend
`ReactiveCrudRepository`, and Spring generates the implementation for you,
just like traditional Spring Data JPA, except fully non-blocking.

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

Built-in methods from `ReactiveCrudRepository` include `findById()`,
`findAll()`, `save()`, `deleteById()`, `count()`, and more — all returning
`Mono`/`Flux`.

## Why It Matters

The repository layer is where the non-blocking nature of your app starts —
every method here returns a `Mono`/`Flux`, so everything built on top of it
(services, controllers) can stay reactive too, without ever needing to
block waiting on the database.
