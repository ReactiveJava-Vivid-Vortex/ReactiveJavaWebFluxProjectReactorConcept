# PUT

## In Simple Terms

A PUT endpoint updates an existing resource. In WebFlux, this typically involves
looking up the existing entity first (to confirm it exists), applying the updates,
and saving — all composed reactively without ever blocking.

## Simple Example

```java
@PutMapping("/products/{id}")
public Mono<ResponseEntity<ProductDto>> updateProduct(
        @PathVariable String id,
        @RequestBody ProductDto dto) {

    return productRepository.findById(id)
        .flatMap(existing -> {
            ProductEntity updated = new ProductEntity(id, dto.name(), dto.price());
            return productRepository.save(updated);
        })
        .map(ProductMapper::toDto)
        .map(ResponseEntity::ok)
        .defaultIfEmpty(ResponseEntity.notFound().build()); // 404 if the id doesn't exist
}
```

Notice the `.flatMap()` here — looking up the existing entity is itself an
asynchronous operation (`findById()` returns a `Mono`), so we need `.flatMap()`
(not `.map()`) to chain the subsequent asynchronous `save()` call.

## Why It Matters

Correctly returning `404 Not Found` when updating a non-existent resource (rather
than silently creating one, or throwing an unhandled exception) is an important REST
API convention — and reactively composing the "check existence, then update" flow
with `.flatMap()` and `.defaultIfEmpty()` is the idiomatic WebFlux way to express it.
