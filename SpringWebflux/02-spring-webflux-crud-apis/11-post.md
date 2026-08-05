# POST

## In Simple Terms

A POST endpoint creates a new resource. In WebFlux, you accept a `@RequestBody`
(either the DTO directly, or wrapped in a `Mono<Dto>` for even more reactive request
handling), save it via the repository, and return the created resource — typically
with a `201 Created` status.

## Simple Example

```java
@PostMapping("/products")
public Mono<ResponseEntity<ProductDto>> createProduct(@RequestBody ProductDto dto) {
    ProductEntity entity = ProductMapper.toEntity(dto);

    return productRepository.save(entity)
        .map(ProductMapper::toDto)
        .map(saved -> ResponseEntity.status(HttpStatus.CREATED).body(saved));
}
```

Using `Mono<ProductDto>` as the request body type instead, for a fully reactive
request-to-response pipeline:

```java
@PostMapping("/products")
public Mono<ResponseEntity<ProductDto>> createProduct(@RequestBody Mono<ProductDto> dtoMono) {
    return dtoMono
        .map(ProductMapper::toEntity)
        .flatMap(productRepository::save)
        .map(ProductMapper::toDto)
        .map(saved -> ResponseEntity.status(HttpStatus.CREATED).body(saved));
}
```

## Why It Matters

Getting the response status right (`201 Created`, not the default `200 OK`) and
returning the fully-saved entity (including any server-generated fields, like an
auto-generated `id`) are the key details that make a POST endpoint behave correctly
according to REST conventions.
