# GET By Id

## In Simple Terms

A "GET By Id" endpoint returns a single resource, implemented using `Mono<T>` and
`repository.findById(id)`. Correctly handling the "not found" case (an empty `Mono`)
is the key detail to get right here.

## Simple Example

```java
@GetMapping("/products/{id}")
public Mono<ResponseEntity<ProductDto>> getProduct(@PathVariable String id) {
    return productRepository.findById(id)
        .map(ProductMapper::toDto)
        .map(ResponseEntity::ok)
        .defaultIfEmpty(ResponseEntity.notFound().build()); // handles "not found" as 404
}
```

A simpler version that relies on WebFlux's default empty-Mono handling (results in
an empty 200 OK body by default unless configured otherwise — being explicit with
`ResponseEntity` as above is usually clearer):

```java
@GetMapping("/products/{id}")
public Mono<ProductDto> getProduct(@PathVariable String id) {
    return productRepository.findById(id).map(ProductMapper::toDto);
}
```

## Why It Matters

Explicitly handling the empty case with `.defaultIfEmpty(ResponseEntity.notFound()...)`
gives you precise control over the HTTP status code returned when a resource doesn't
exist — an important detail for building a well-behaved, predictable REST API.
