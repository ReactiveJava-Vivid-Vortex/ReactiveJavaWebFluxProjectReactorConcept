# DELETE

## In Simple Terms

A DELETE endpoint removes a resource. In WebFlux, `repository.deleteById(id)`
returns `Mono<Void>` — there's no real "value" to hand back, just a signal
that the operation finished, one way or the other.

## Simple Example

```java
@DeleteMapping("/products/{id}")
public Mono<ResponseEntity<Void>> deleteProduct(@PathVariable String id) {
    return productRepository.findById(id)
        .flatMap(existing -> productRepository.delete(existing)
            .then(Mono.just(ResponseEntity.noContent().<Void>build())) // 204 No Content
        )
        .defaultIfEmpty(ResponseEntity.notFound().build()); // 404 if it didn't exist
}
```

A simpler version, skipping the existence check (returns 204 either way):

```java
@DeleteMapping("/products/{id}")
@ResponseStatus(HttpStatus.NO_CONTENT)
public Mono<Void> deleteProduct(@PathVariable String id) {
    return productRepository.deleteById(id);
}
```

## Why It Matters

Whether to return `404` for deleting something that isn't there, versus
always returning `204 No Content` regardless (a common, simpler
convention — deleting something already gone is often treated as a
harmless success), is a deliberate design choice. Both are common in
production; being consistent across your whole API matters more than which
one you pick.
