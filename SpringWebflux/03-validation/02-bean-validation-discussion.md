# Bean Validation Discussion

## In Simple Terms

Bean Validation (`@NotNull`, `@Size`, `@Min`, and friends) is the standard
annotation-based validation style you already know from traditional Spring
MVC. In Spring WebFlux, those same annotations still work on your DTOs, but
there's a catch worth knowing: automatic `@Valid` triggering doesn't always
play nicely with reactive request bodies (`Mono<Dto>`) the way it does with
plain objects — this trips people up a lot.

## Simple Example

```java
public record ProductDto(
    @NotBlank(message = "Name is required") String name,
    @Positive(message = "Price must be positive") double price
) {}
```

Works cleanly with a plain (non-`Mono`) request body:

```java
@PostMapping
public Mono<ProductDto> create(@Valid @RequestBody ProductDto dto) {
    // @Valid triggers automatically here, since the body is a plain object
    return productService.create(dto);
}
```

**Watch out for this:** `@Valid` doesn't automatically kick in when the
request body is wrapped in `Mono<ProductDto>` — in that case you'd need to
validate manually inside the reactive chain (say, using a `Validator` bean
directly).

## Why It Matters

Knowing this nuance saves you from a subtle, easy-to-miss bug: assuming
`@Valid @RequestBody Mono<ProductDto>` validates automatically (it often
doesn't, depending on your Spring version and setup), when you actually
need to validate explicitly. Always double-check with a test that
validation is really firing.
