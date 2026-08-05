# GET All

## In Simple Terms

A "GET All" endpoint returns the full collection of a resource — implemented in
WebFlux by returning a `Flux<T>` (or `Flux<Dto>`) from `repository.findAll()`,
optionally mapped, filtered, or paginated.

## Simple Example

```java
@GetMapping("/products")
public Flux<ProductDto> getAllProducts() {
    return productRepository.findAll()
        .map(ProductMapper::toDto);
}
```

With optional query-parameter filtering:

```java
@GetMapping("/products")
public Flux<ProductDto> getAllProducts(
        @RequestParam(required = false) String category) {
    Flux<ProductEntity> source = category != null
        ? productRepository.findByCategory(category)
        : productRepository.findAll();

    return source.map(ProductMapper::toDto);
}
```

## Why It Matters

"GET All" is usually the first CRUD endpoint written for any resource, and it
establishes the pattern (`Flux` return type, entity-to-DTO mapping) that the rest of
your reactive CRUD endpoints will follow.
