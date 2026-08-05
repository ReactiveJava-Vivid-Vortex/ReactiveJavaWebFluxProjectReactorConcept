# Entity Mapping

## In Simple Terms

"Entity mapping" is the process of converting between your database `Entity`
(annotated for R2DBC/persistence) and your API-facing `DTO`, in both directions —
`Entity -> DTO` for responses, and `DTO -> Entity` for incoming create/update
requests. In reactive code, this mapping happens inside `.map()` calls within your
reactive pipeline.

## Simple Example

```java
@Table("products")
public record ProductEntity(@Id String id, String name, double cost) {}

public record ProductDto(String id, String name, double price) {}

public class ProductMapper {
    public static ProductDto toDto(ProductEntity entity) {
        return new ProductDto(entity.id(), entity.name(), entity.cost());
    }

    public static ProductEntity toEntity(ProductDto dto) {
        return new ProductEntity(dto.id(), dto.name(), dto.price());
    }
}
```

Using the mapper inside a reactive pipeline:

```java
@GetMapping("/{id}")
public Mono<ProductDto> getProduct(@PathVariable String id) {
    return repository.findById(id)
        .map(ProductMapper::toDto);
}

@PostMapping
public Mono<ProductDto> create(@RequestBody ProductDto dto) {
    ProductEntity entity = ProductMapper.toEntity(dto);
    return repository.save(entity)
        .map(ProductMapper::toDto);
}
```

## Why It Matters

Keeping entity-to-DTO mapping explicit and centralized (e.g., in a dedicated
`Mapper` class) avoids scattering ad-hoc conversion logic throughout your
controllers and services — making it easier to keep your API contract stable even as
your internal entity structure evolves.
