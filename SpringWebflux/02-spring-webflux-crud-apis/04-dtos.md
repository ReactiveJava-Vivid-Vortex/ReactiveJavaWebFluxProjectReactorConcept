# DTOs

## In Simple Terms

A DTO (Data Transfer Object) is a plain object shaped specifically for
what your API shows to the outside world — different from your internal
database `Entity`. In WebFlux, you typically `.map()` an `Entity` (loaded
via R2DBC, say) into a DTO before sending it back from a controller, so you
control exactly what fields get exposed, independent of your database
structure.

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

Notice `internalSku` from the entity never makes it into the API response
— the DTO deliberately leaves it out.

## Why It Matters

DTOs keep your public API separate from your internal database structure —
you can change how data is stored (renaming columns, restructuring tables)
without breaking anyone using your API, and you avoid accidentally leaking
internal-only fields (like `internalSku`) to the outside world.
