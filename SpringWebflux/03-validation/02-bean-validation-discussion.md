# Bean Validation Discussion

## In Simple Terms

**Bean Validation** (JSR 380, e.g., `@NotNull`, `@Size`, `@Min`) is the standard
Java annotation-based validation approach, well-established in traditional Spring
MVC. In Spring WebFlux, Bean Validation annotations still work on your DTOs, but
there's an important nuance: automatic `@Valid` triggering doesn't always integrate
as seamlessly with reactive types (`Mono<Dto>` request bodies) as it does with plain
objects — this is a commonly discussed WebFlux gotcha.

## Simple Example

```java
public record ProductDto(
    @NotBlank(message = "Name is required") String name,
    @Positive(message = "Price must be positive") double price
) {}
```

Works cleanly with a plain (non-Mono) request body:

```java
@PostMapping
public Mono<ProductDto> create(@Valid @RequestBody ProductDto dto) {
    // @Valid triggers automatically here, since the body is a plain object
    return productService.create(dto);
}
```

**Gotcha:** `@Valid` does **not** automatically apply when the request body itself is
wrapped in `Mono<ProductDto>` — you'd need to manually validate inside the reactive
chain in that case (e.g., using a `Validator` bean directly).

## Why It Matters

Understanding this nuance avoids a subtle, hard-to-notice bug: assuming
`@Valid @RequestBody Mono<ProductDto>` validates automatically (it often does NOT,
depending on Spring version and configuration), when in practice you may need
explicit validation logic. Always verify validation actually triggers with a test.
