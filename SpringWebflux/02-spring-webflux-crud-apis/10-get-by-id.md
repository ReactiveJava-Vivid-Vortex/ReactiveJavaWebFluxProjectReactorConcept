# GET By Id

## In Simple Terms

A "GET By Id" endpoint returns a single resource, built with `Mono<T>` and
`repository.findById(id)`. The one detail worth getting right here is how
you handle the "not found" case — an empty `Mono`.

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

A simpler version that just relies on WebFlux's default empty-`Mono`
handling (results in an empty `200 OK` body by default unless you
configure it otherwise — being explicit with `ResponseEntity`, like above,
is usually clearer):

```java
@GetMapping("/products/{id}")
public Mono<ProductDto> getProduct(@PathVariable String id) {
    return productRepository.findById(id).map(ProductMapper::toDto);
}
```

## Why It Matters

Explicitly handling the empty case with
`.defaultIfEmpty(ResponseEntity.notFound()...)` gives you precise control
over what status code gets returned when a resource doesn't exist — an
important detail for building a predictable, well-behaved API.
