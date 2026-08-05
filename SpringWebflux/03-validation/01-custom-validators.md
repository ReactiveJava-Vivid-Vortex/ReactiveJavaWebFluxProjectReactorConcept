# Custom Validators

## In Simple Terms

A custom validator is your own class/logic that checks incoming data against
business rules that go beyond simple annotations (like `@NotNull`) — e.g., "this
email must not already exist," or "the discount code must be currently active." In a
reactive pipeline, custom validation often needs to be asynchronous itself (e.g.,
checking a database), so it's expressed as a `Mono`-returning method.

## Simple Example

```java
@Component
public class ProductValidator {

    private final ProductRepository repository;

    public Mono<Void> validate(ProductDto dto) {
        if (dto.price() <= 0) {
            return Mono.error(new ValidationException("Price must be positive"));
        }

        return repository.existsByName(dto.name())
            .flatMap(exists -> exists
                ? Mono.error(new ValidationException("Product name already exists"))
                : Mono.empty()
            );
    }
}
```

Using it in a service method:

```java
public Mono<ProductDto> createProduct(ProductDto dto) {
    return validator.validate(dto)
        .then(Mono.defer(() -> repository.save(ProductMapper.toEntity(dto))))
        .map(ProductMapper::toDto);
}
```

## Why It Matters

Custom validators let you express business rules that simple declarative annotations
can't express — especially rules requiring a database lookup or external check —
while keeping validation logic fully reactive and non-blocking, consistent with the
rest of your WebFlux pipeline.
