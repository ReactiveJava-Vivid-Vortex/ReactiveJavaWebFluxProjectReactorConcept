# An Empty Mono Does NOT Automatically Mean 404

## In Simple Terms

A very common assumption: "if my controller returns an empty `Mono`, WebFlux will
automatically send back a `404 Not Found`." **This is not automatically true.** By
default, if an annotated `@RestController` method returns a `Mono<T>` that
completes empty, Spring WebFlux responds with **`200 OK` and an empty body** — not
`404`. You have to make the "empty means not found" decision **explicitly**.

## Simple Example

```java
// This does NOT automatically return 404 when the product doesn't exist!
@GetMapping("/products/{id}")
public Mono<ProductDto> getProduct(@PathVariable String id) {
    return productRepository.findById(id).map(ProductMapper::toDto);
    // If findById() completes empty -> client gets 200 OK with an empty body
}
```

To actually get a `404`, you must explicitly convert the empty case into a
`ResponseEntity.notFound()` (or throw/emit a custom exception handled by
`@ControllerAdvice` — see the Reactive Error Handling topic):

```java
@GetMapping("/products/{id}")
public Mono<ResponseEntity<ProductDto>> getProduct(@PathVariable String id) {
    return productRepository.findById(id)
        .map(ProductMapper::toDto)
        .map(ResponseEntity::ok)
        .defaultIfEmpty(ResponseEntity.notFound().build()); // NOW it's explicit
}
```

## Why It Matters

Relying on an assumed default here is a genuinely common production bug: an API
that silently returns `200 OK` with an empty body for missing resources, instead of
a proper `404`, confuses API consumers and breaks REST conventions. Always be
explicit about the empty case — either with `.defaultIfEmpty(ResponseEntity...)` as
shown, or by turning "not found" into a custom exception via `.switchIfEmpty(Mono.error(...))`
and letting a `@ControllerAdvice` handle it centrally.
