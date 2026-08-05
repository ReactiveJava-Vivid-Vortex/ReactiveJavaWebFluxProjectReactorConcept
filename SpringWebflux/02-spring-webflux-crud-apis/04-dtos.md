# DTOs

## In Simple Terms

A **DTO (Data Transfer Object)** is a plain object shaped specifically for what an
API exposes to clients — distinct from your internal database `Entity`. In WebFlux,
you typically `.map()` an `Entity` (e.g., loaded via R2DBC) into a DTO before
returning it from a controller, so you can control exactly what fields are exposed,
independent of your database schema.

## Simple Example

```java
// Entity - matches the database table structure
public record ProductEntity(String id, String name, double cost, String internalSku) {}

// DTO - only what the API should expose
public record ProductDto(String id, String name, double price) {}
```

```java
@GetMapping("/{id}")
public Mono<ProductDto> getProduct(@PathVariable String id) {
    return productRepository.findById(id)
        .map(entity -> new ProductDto(entity.id(), entity.name(), entity.cost()));
}
```

Notice `internalSku` from the entity never reaches the API response — the DTO
deliberately excludes it.

## Why It Matters

Using DTOs decouples your public API contract from your internal database schema —
you can change how data is stored (renaming columns, restructuring tables) without
breaking API consumers, and you avoid accidentally leaking internal-only fields
(like `internalSku`) to external clients.
