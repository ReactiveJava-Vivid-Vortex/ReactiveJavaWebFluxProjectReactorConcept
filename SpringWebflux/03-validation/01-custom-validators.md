# Custom Validators

## In Simple Terms

A custom validator is your own logic for checking incoming data against
rules that go beyond a simple annotation like `@NotNull` — things like
"this email can't already be taken," or "the discount code has to be
currently active." In a reactive app, custom validation often needs to be
async itself (checking a database, say), so it's written as a
`Mono`-returning method.

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

Custom validators let you express rules that simple annotations just
can't — especially ones needing a database check or an external lookup —
while keeping validation itself fully reactive and non-blocking, in step
with the rest of your WebFlux pipeline.
