# Service Layer

## In Simple Terms

The service layer contains your application's **business logic** — validation,
orchestration of multiple repository/external calls, and entity-to-DTO
transformation — sitting between the controller (HTTP concerns) and the repository
(data access concerns). In reactive applications, service methods also return
`Mono`/`Flux`, composing the repository layer's reactive results with additional
logic.

## Simple Example

```java
@Service
public class ProductService {

    private final ProductRepository repository;

    public ProductService(ProductRepository repository) {
        this.repository = repository;
    }

    public Mono<ProductDto> getProduct(String id) {
        return repository.findById(id)
            .switchIfEmpty(Mono.error(new ProductNotFoundException(id)))
            .map(ProductMapper::toDto);
    }

    public Mono<ProductDto> createProduct(ProductDto dto) {
        if (dto.price() <= 0) {
            return Mono.error(new IllegalArgumentException("Price must be positive"));
        }
        ProductEntity entity = ProductMapper.toEntity(dto);
        return repository.save(entity).map(ProductMapper::toDto);
    }
}
```

## Why It Matters

Keeping business logic in a dedicated service layer (rather than directly in
controllers) keeps controllers thin and focused purely on HTTP concerns
(request/response mapping), while the service layer remains independently testable
and reusable — the same layering principle from traditional Spring applications,
just expressed reactively.
