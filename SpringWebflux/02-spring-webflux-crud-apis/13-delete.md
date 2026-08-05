# DELETE

## In Simple Terms

A DELETE endpoint removes a resource. In WebFlux, `repository.deleteById(id)`
returns `Mono<Void>` — there's no meaningful "value" to return, just a completion
signal indicating the operation finished (successfully or with an error).

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

A simpler version, without checking existence first (returns 204 either way):

```java
@DeleteMapping("/products/{id}")
@ResponseStatus(HttpStatus.NO_CONTENT)
public Mono<Void> deleteProduct(@PathVariable String id) {
    return productRepository.deleteById(id);
}
```

## Why It Matters

Whether to return `404` for deleting a non-existent resource, versus always
returning `204 No Content` regardless (a common, simpler REST convention — deleting
something that's already gone is often considered a no-op success), is a deliberate
API design decision. Both patterns are common in production; consistency across your
API matters more than which one you pick.
