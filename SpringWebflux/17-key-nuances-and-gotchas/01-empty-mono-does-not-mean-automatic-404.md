# An Empty Mono Does NOT Automatically Mean 404

## In Simple Terms

A really common assumption: "if my controller returns an empty `Mono`,
WebFlux will just send back a `404 Not Found` automatically." That's not
true. By default, if an annotated `@RestController` method returns a
`Mono<T>` that finishes empty, WebFlux responds with `200 OK` and an empty
body — not `404`. You have to decide "empty means not found" yourself,
explicitly.

## Simple Example

```java
// This does NOT automatically return 404 when the product doesn't exist!
@GetMapping("/products/{id}")
public Mono<ProductDto> getProduct(@PathVariable String id) {
    return productRepository.findById(id).map(ProductMapper::toDto);
    // If findById() completes empty -> client gets 200 OK with an empty body
}
```

To actually get a `404`, you have to explicitly turn the empty case into
a `ResponseEntity.notFound()` (or throw/emit a custom exception that a
`@ControllerAdvice` handles — see the Reactive Error Handling topic):

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

Relying on this assumed default is a genuinely common production bug: an
API silently returning `200 OK` with an empty body for missing resources
instead of a proper `404` confuses whoever's calling it and breaks REST
conventions. Always spell out the empty case yourself — either with
`.defaultIfEmpty(ResponseEntity...)` as shown, or by turning "not found"
into a custom exception with `.switchIfEmpty(Mono.error(...))` and letting
a `@ControllerAdvice` handle it centrally.
