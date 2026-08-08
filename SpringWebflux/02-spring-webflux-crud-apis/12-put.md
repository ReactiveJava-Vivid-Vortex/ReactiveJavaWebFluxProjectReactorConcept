# PUT

## In Simple Terms

A PUT endpoint updates an existing resource. In WebFlux, this usually means
looking up the existing entity first (to confirm it exists), applying the
changes, and saving — all done reactively, without ever blocking.

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

Notice the `.flatMap()` here — looking up the existing entity is itself
async (`findById()` returns a `Mono`), so you need `.flatMap()` (not
`.map()`) to chain the follow-up async `save()` call.

## Why It Matters

Correctly returning `404 Not Found` when updating a resource that doesn't
exist (instead of quietly creating one, or throwing an unhandled
exception) is an important REST convention — and composing the "check it
exists, then update" flow reactively with `.flatMap()` and
`.defaultIfEmpty()` is the natural WebFlux way to write it.
