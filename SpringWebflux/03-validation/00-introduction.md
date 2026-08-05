# Validation — Topic Overview

## What Is This Topic About? (In Simple Terms)

Validation makes sure incoming request data is well-formed and meets your business
rules **before** it reaches your service/repository logic. In WebFlux, most of this
looks exactly like traditional Spring — Bean Validation annotations
(`@NotBlank`, `@Positive`) on your DTO fields, triggered by `@Valid`:

```java
public record ProductDto(
    @NotBlank String name,
    @Positive double price
) {}

@PostMapping
public Mono<ProductDto> create(@Valid @RequestBody ProductDto dto) {
    return productService.create(dto);
}
```

**The one important WebFlux-specific gotcha:** `@Valid` reliably triggers on a
plain `@RequestBody ProductDto`, but does **not** automatically trigger if the body
is wrapped as `Mono<ProductDto>` — a subtle trap that's easy to miss until a bad
request slips through untested.

Beyond simple field checks, some rules need a database lookup (e.g., "this email
must not already exist") — these can't be expressed as a simple annotation, since
they're inherently asynchronous. That's where **custom validators** come in,
returning a `Mono<Void>` that you `.then()` before proceeding:

```java
public Mono<Void> validate(ProductDto dto) {
    return repository.existsByName(dto.name())
        .flatMap(exists -> exists
            ? Mono.error(new ValidationException("Name already exists"))
            : Mono.empty());
}
```

## Quick Revision Cheat Sheet

| # | Concept | One-Line Summary |
|---|---|---|
| 1 | **Custom Validators** | Your own async validation logic (e.g., DB lookups) — expressed as a `Mono<Void>`-returning method. |
| 2 | **Bean Validation discussion** | `@NotBlank`/`@Positive` etc. still work, but `@Valid` doesn't auto-trigger on a `Mono<Dto>` request body — verify with a test! |
| 3 | **DTO validation** | Annotate DTO fields directly so malformed requests are caught before reaching business logic. |
| 4 | **Validation inside reactive pipelines** | Weave async validation into the chain with `.flatMap()`/`Mono.error()`, so a failure short-circuits the rest. |

## How It All Fits Together

```
Incoming request body
      │
      ▼
Simple field rules (NotBlank, Positive, ...)  ──▶ @Valid @RequestBody Dto  (works)
                                                    @Valid @RequestBody Mono<Dto> (does NOT auto-trigger!)
      │
      ▼
Business rules needing a DB/external check  ──▶ custom Mono<Void> validator, chained with .then()
      │
      ▼
Passes all checks → proceed to service logic
Fails any check    → Mono.error(...) short-circuits, handled by Reactive Error Handling (next topic)
```

Remember the golden rule of this topic: **whenever your request body is wrapped as
`Mono<Dto>`, double-check that `@Valid` is actually firing** — don't assume it works
just because it looks the same as Spring MVC.
